---
layout: post
title: Python NLTK + Stanford NLP
tags: [python, nlp]
categories: [development]
---

Как известно, в Python стандартом работы с натуральным языком де-факто является NLTK. Несмотря на это, я довольно долго использовал Pattern от CLiPS из-за его простоты и скорости (многие отмечают тормознутость NLTK).

Но наступил момент, когда почти вся кодовая база была успешно портирована на Python 3.5, а разработчики Pattern так и не сделали версию с поддержкой третьей версии. И, судя по всему, не собираются.

Что-ж, будем использовать NLTK. От него мне нужны: токенизация, выделение POS (part-of-speech), получение N-grams и классификация твитов на группы с использованием Naive Bayes. Все это дело на Python 3.5.

{% highlight bash %}
pip3 install nltk
{% endhighlight %}

Модули для NLTK устанавливаются так:

{% highlight python %}
import nltk
nltk.download()
{% endhighlight %}

... или так:

{% highlight bash %}
python3 -m nltk.downloader punkt averaged_perceptron_tagger
# python3 -m nltk.downloader all
{% endhighlight %}

Теперь немного примеров.

{% highlight python %}
tweet = '''
    Kinto by Mozilla - An open source Parse alternative >>
    https://github.com/Kinto/kinto/ #python #parse
'''

# Получаем токены. Стандартный универсальный метод:
from nltk import word_tokenize
print(word_tokenize(tweet))
'''
[
    'Kinto', 'by', 'Mozilla', '-', 'An', 'open', 'source',
    'Parse', 'alternative','>', '>', 'https', ':',
    '//github.com/Kinto/kinto/', '#', 'python', '#', 'parse'
]
'''

# С использованием специализированного класса
from nltk.tokenize import TweetTokenizer
# Убрать имена пользователей и многократно повторяемые символы
tt = TweetTokenizer(preserve_case=True, reduce_len=False, strip_handles=False)
print(tt.tokenize(tweet))
'''
[
    'Kinto', 'by', 'Mozilla', '-', 'An', 'open',
    'source', 'Parse', 'alternative', '>', '>',
    'https://github.com/Kinto/kinto/', '#python', '#parse'
]
'''

# Возьмем немного почищеные токены. Воспользуемся обычным токенизатором
tokens = [
    t for t in word_tokenize(tweet)
        if len(t) > 1 and not t.startswith('http')
        and not t.startswith('/')
]
'''
[
    'Kinto', 'by', 'Mozilla', 'An', 'open', 'source',
    'Parse', 'alternative', 'python', 'parse'
]
'''
{% endhighlight %}

Токены получили, теперь будем узнавать части речи. Можно использовать встроенный метод, а можно пойти интересным путем, и привлечь Stanford NLP библиотеки. Для этого скачаем и распакуем каталог с ними в нашу рабочую директорию, где пишем код. В Python укажем место расположения JAR и модулей через переменные окружения.

{% highlight python %}
import os
from nltk.tag import StanfordPOSTagger
os.environ['CLASSPATH'] = os.path.join(
    os.path.curdir, 'stanford-postagger-2015-04-20'
)
os.environ['STANFORD_MODELS'] = os.path.join(
    os.path.curdir, 'stanford-postagger-2015-04-20', 'models'
)
stanford_tagger = StanfordPOSTagger('english-bidirectional-distsim.tagger')
{% endhighlight %}

Теперь можно сравнить результаты методов:

{% highlight python %}
from nltk import pos_tag
print(pos_tag(tokens))
print(stanford_tagger.tag(tokens))
{% endhighlight %}

Результаты почти идентичны, за исключением "An". StanfordPOSTagger считает его существительным. Допустим, мы выберем последний вариант. Найдем все имена существительные (NN, NNS, NNP, NNP-PERS, NNP-ORG):

{% highlight python %}
tagger = stanford_tagger.tag
nouns = [word.lower() for word, pos in tagger(tokens) if pos.startswith('NN')]
print(nouns)  # ['kinto', 'mozilla', 'an', 'source', 'parse', 'python', 'parse']
{% endhighlight %}

Теперь N-grams. Не проблема сделать самостоятельно, но в NLTK уже есть пара готовых функций.

{% highlight python %}
from nltk.util import everygrams, ngrams
print(list(ngrams(nouns, 2)))
# [('kinto', 'mozilla'), ('mozilla', 'an'), ...  ('python', 'parse')]
print(list(everygrams(nouns, min_len=2, max_len=3)))
'''
[
    ('kinto', 'mozilla'), ('mozilla', 'an'), ... ('python', 'parse'),
    ('kinto', 'mozilla', 'an'), ('mozilla', 'an', 'source'),
    ... ('parse', 'python', 'parse')
]
'''
{% endhighlight %}

Как я уже говорил, NLTK порой очень медленный. С использованием Stanford библиотек он медленнее в разы. Существенно улучшает ситуацию обработка больших объемов твитов разом, а не один за другим.

{% highlight python %}
from nltk import pos_tag, pos_tag_sents
tokens_group = [tokens for i in range(100)]
print(pos_tag_sents(tokens_group))  # 0.3 sec
print(stanford_tagger.tag_sents(tokens_group))  # 6.4 sec !!
{% endhighlight %}

На сём откланиваюсь.