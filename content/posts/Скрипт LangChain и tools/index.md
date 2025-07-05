---
date: 2025-07-05T16:00:07+03:00
description: Создание ИИ-агента на Python с помощью библиотеки LangChain
draft: false
slug: ai-agent-python-with-langchain-and-tools
tags:
    - ai
    - python
title: Создание ИИ-агента на Python
---

ИИ шагает по планете с такой скоростью, что это пугает. Но лучше не пугаться, а постараться вспрыгнуть на эту волну и прокатиться на ней.
Самое популярное, с чем мы сталкиваемся каждый день - это LLM (Large Language Model), говорящие ИИ-боты. Они много что могут нам рассказать, но вот ручек и ножек им не завезли.
Другое дело - ИИ-агенты, это по сути те-же LLM, но с ручками и ножками. То есть с инструментами.
Имея доступ к LLM (публичной или локальной) можно своими руками приделать им недостающие конечности и соорудить маленького, но своего Джарвиса.
Я продемонстрирую, как это сделать с помощью Python и библиотеки [LangChain](https://www.langchain.com).

Сначала сделаем самого агента. Я использовал локальную LLM, запущенную в [LM Studio](https://lmstudio.ai), но можно также и публичную любую прицепить (если достанете API-key к ней).

```python
import uuid
from typing import Sequence

# Библиотеки LangChain
from langchain_core.language_models import LanguageModelLike
from langchain_core.runnables import RunnableConfig
from langchain_core.tools import BaseTool
from langchain_openai import ChatOpenAI
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.prebuilt import create_react_agent

# Инструменты для LLM, их создадим в отдельном модуле
from tools import add_furniture, del_furniture, get_furniture, list_furniture

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

    def invoke(self, content: str, temperature: float = 0.1) -> str:
        """Отправляет сообщение в чат"""
        message: dict = {"role": "user", "content": content}
        response = self._agent.invoke(
            {"messages": [message], "temperature": temperature},
            config=self._config,
        )
        return response["messages"][-1].content


def print_agent_response(llm_response: str) -> None:
    """Чтоб вывод агента был красивее и отличался от остального"""
    print(f"\033[35mИИ-агент: {llm_response}\033[0m")


def get_user_prompt() -> str:
    return input("\nПользователь: ")


def config_master():
    agent = LLMAgent(
	    # Перечисляем доступные инструменты
        model, tools=[add_furniture, del_furniture, get_furniture, list_furniture]
    )
    # Системный промпт - задает базовый контекст LLM-ке. Важно описать в деталях.
    system_prompt = (
        "Ты управляющий мебелью в частном доме. "
        "Ты умеешь добавлять предметы мебели, получать их по идентификатору, "
        "удалять их по идентификатору, а также показывать их список с фильтром по имени. "
        "Чтобы добавить мебель, тебе нужно узнать ее идентификатор и название. "
        "Чтобы получить или удалить предмет мебели, тебе нужно узнать его идентификатор. "
        "Чтобы искать мебель, тебе нужно узнать ее название. "
        "Для добавления новой мебели у тебя нет значений по умолчанию, поэтому "
        "узнай все необходимые данные у пользователя. "
        "Строго спроси все необходимые поля, ничего не придумывай сам. "
        "При поиске мебели по имени ответ выводи в виде маркированного списка. "
        "Всегда отвечай строго на русском языке."
    )

    agent_response = agent.invoke(
        content=system_prompt,
    )

	# В цикле задаем вопросы и выводим ответы
    while True:
        print_agent_response(agent_response)
        agent_response = agent.invoke(get_user_prompt())


if __name__ == "__main__":
    try:
        config_master()

    except KeyboardInterrupt:
        print()
        print("Завершение работы")
```

Теперь реализуем сами инструменты

```python
from dataclasses import dataclass
from typing import List, Optional

from langchain_core.tools import tool


@dataclass
class Furniture:
    """Предмет мебели"""

    id: int  # Идентификатор предмета мебели
    name: str  # Название предмета мебели


class Storage:
	"""Хранилище для мебели, простенькое"""
    def __init__(self) -> None:
        self._data = {}

    def add(self, furn: Furniture):
        self._data[furn.id] = furn
        print(self._data)

    def delete(self, id: int):
        if self.get(id):
            del self._data[id]
        print(self._data)

    def get(self, id: int) -> Optional[Furniture]:
        return self._data.get(id, None)

    def list(self, name) -> List[Furniture]:
        return [furn for furn in self._data.values() if name in furn.name]


storage = Storage()


@tool
def add_furniture(furn: Furniture) -> Furniture:
    """
    Добавление предмета мебели

    Args:
        furn (Furniture): предмет мебели

    Returns:
        Furniture
    """
    storage.add(furn)
    return furn


@tool
def del_furniture(id: int) -> bool:
    """
    Удаление предмета мебели по идентификатору

    Args:
        id (int): идентификатор предмета мебели

    Returns:
        bool
    """
    storage.delete(id)
    return True


@tool
def get_furniture(id: int) -> Optional[Furniture]:
    """
    Получение предмета мебели по идентификатору

    Args:
        id (int): идентификатор предмета мебели

    Returns:
        Optional[Furniture]
    """
    return storage.get(id)


@tool
def list_furniture(name: str) -> List[Furniture]:
    """
    Получение списка мебели по названию

    Args:
        id (int): идентификатор предмета мебели

    Returns:
        List[Furniture]
    """
    return storage.list(name)
```

На этом всё. Теперь у вас есть представление, как собрать себе умную игрушку. В качестве инструментов можно выдать агенту методы доступа к файловой системе, чтоб он мог читать и писать, или инструменты отправки email, или запросов в интернет. Дерзайте.