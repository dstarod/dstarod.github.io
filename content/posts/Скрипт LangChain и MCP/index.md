---
date: 2025-07-09T15:51:32+03:00
description: Создание MCP-сервера и его использование ИИ-агентом на Python с библиотеками LangChain и FastMCP
draft: false
slug: mcp-server-ai-agent-python
tags:
    - python
    - ai
title: MCP-сервер для ИИ-агента
---

Совсем недавно я рассказывал о том, [как создать ИИ-агента и дать ему в руки инструменты](https://dstarod.github.io/posts/ai-agent-python-with-langchain-and-tools/), чтобы взаимодействовать с реальным миром. Сегодня я покажу пример, как сделать шаг вперед - создать MCP-сервер, который будет предоставлять инструменты для LLM в согласии с современными стандартами отрасли. О том, что это такое и откуда у него растут ножки, можно почитать в интернетах, или [вот вам собранная Perplexity статья](https://www.perplexity.ai/search/212a11a5-84dd-4f7b-b697-f1162384c2a3).

Код агента почти не поменялся, за исключением того, что инструменты мы напрямую не передаем, а получаем их от MCP-сервера.

```python
import asyncio
import logging
import uuid
from typing import Sequence

# Библиотеки LangChain
from langchain_core.language_models import LanguageModelLike
from langchain_core.runnables import RunnableConfig
from langchain_core.tools import BaseTool
from langchain_mcp_adapters.client import MultiServerMCPClient
from langchain_openai import ChatOpenAI
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.prebuilt import create_react_agent

# Спрячем отладочные логи библиотек
logging.getLogger("httpx").setLevel(logging.WARNING)
logging.getLogger("mcp").setLevel(logging.WARNING)

# Создаем объект модельки
model = ChatOpenAI(
    base_url="http://localhost:1234/v1",
    model="gemma-3-12b",
    api_key="Not needed",
    temperature=0.1,
    max_tokens=400,
    max_retries=3,
)


class LLMAgent:
    def __init__(self, model: LanguageModelLike, tools: Sequence[BaseTool]):
        self._model = model
        # Создаем объект агента, передаем инструменты и запоминалку контекста
        self._agent = create_react_agent(
            model, tools=tools, checkpointer=InMemorySaver()
        )
        self._config: RunnableConfig = {
            "configurable": {"thread_id": uuid.uuid4().hex},
            "recursion_limit": 100,
        }

    async def ainvoke(self, content: str, temperature: float = 0.1) -> str:
        """Отправляет сообщение в чат"""
        message: dict = {"role": "user", "content": content}
        response = await self._agent.ainvoke(
            {"messages": [message], "temperature": temperature},
            config=self._config,
        )
        return response["messages"][-1].content


def print_agent_response(llm_response: str) -> None:
	"""Чтоб вывод агента был красивее и отличался от остального"""
    print(f"\033[35mAgent:\033[0m {llm_response}")


def get_user_prompt() -> str:
    return input("\033[33mUser:\033[0m ")


async def file_manager():
	# Создаем клиент для обращения к MCP-серверу
    client = MultiServerMCPClient(
        {
            "localhost": {
                "url": "http://localhost:8000/mcp/",
                "transport": "streamable_http",
            }
        }
    )
	# Получаем доступные инструменты
    tools = await client.get_tools()

	# Создаем агента с передачей ему MCP-инструментов
    agent = LLMAgent(model, tools=tools)
	# Системный промпт - задает базовый контекст LLM-ке. Важно описать в деталях.
    system_prompt = (
        "Ты универсальный помощник редактирования файлов. "
        "Ты умеешь читать содержимое файлов по полному пути. "
        "Ты умеешь вносить изменения в файл, зная полный путь к нему и содержимое файла. "
        "Для работы с файлами используй предоставленные инструменты. "
        "Всегда отвечай на русском языке. "
        "Если чего-то не знаешь, скажи Я не знаю. "
    )

    agent_response = await agent.ainvoke(system_prompt)

	# В цикле задаем вопросы и выводим ответы
    while True:
        print_agent_response(agent_response)
        agent_response = await agent.ainvoke(get_user_prompt())


if __name__ == "__main__":
    try:
        loop = asyncio.new_event_loop()
        loop.run_until_complete(file_manager())

    except Exception as e:
        print(e)

    except KeyboardInterrupt:
        print()
        print("Завершение работы")
```

Ну а теперь собственно MCP-сервер. Зацените, как просто. Здесь главное - описание инструментов, входящих и исходящих параметров, которые позволят LLM-ке понять, в каких случаях какой инструмент вызывать.

```python
from mcp.server import FastMCP
import logging

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

mcp = FastMCP("Доступ к файловой системе")


@mcp.tool(description="Чтение файла по полному пути")
def read_file(path: str) -> str:
    """
    Читает содержимое файла по указанному пути.

    Args:
        path (str): Полный путь к файлу.

    Returns:
        str: Содержимое файла или сообщение об ошибке, если файл не найден или нет прав доступа.
    """
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        logging.error(f"Файл не найден: {path}")
        return f"Ошибка: Файл не найден: {path}"
    except Exception as e:
        logging.error(f"Ошибка при чтении файла {path}: {e}")
        return f"Ошибка: Не удалось прочитать файл: {e}"


@mcp.tool(description="""
Запись данных в файл
Args:
    path (str): File path, полный путь к файлу
    content (str): File content, содержимое файла
Returns:
    bool
""")
def write_file(path: str, content: str) -> bool:
    """
    Записывает данные в файл по указанному пути.

    Args:
        path (str): Полный путь к файлу.
        content (str): Содержимое файла для записи.

    Returns:
        bool: True, если запись прошла успешно, False в противном случае.
    """
    try:
        with open(path, 'w') as f:
            f.write(content)
        logging.info(f"Успешно записано в файл: {path}")
        return True
    except Exception as e:
        logging.error(f"Ошибка при записи в файл {path}: {e}")
        return False


if __name__ == "__main__":
    mcp.run(transport="streamable-http")
```

Теперь вы прям на пике прогресса. Можете сделать агента, который будет использовать инструменты, запущенные отдельно (например на сервере с БД) по самым современным стандартам.