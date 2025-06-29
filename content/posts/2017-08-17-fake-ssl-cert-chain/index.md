---
title: Создание валидной цепочки SSL сертификатов
tags:
- ssl
- memo
- python
slug: fake-ssl-cert-chain
date: 2017-08-17 00:00:00+03:00
draft: false
---

Для написания тестов на валидность загружаемых сертификатов мне потребовалось создать несколько вариантов
цепочек и приватных ключей. Прошерстив некоторый объем интернета, я собрал всю необходимую информацию о том,
как локально сгенерировать корректную цепочку SSL сертификатов, которая будет проходить проверку стандартными
средствами.

Чтобы не писать кучу команд всякий раз, напишем bash-скрипт. Для начала, зададим несколько настроечных констант:

```bash
CSR_FILE="csrfile.csr"
KEY_BITS=2048
CONF_DIR="conf"
CHAIN_CRT="ca_chain.crt"
```

Затем пара функций для подготовки и последующей подчистки каталога с настройками: 

```bash
function clean {
    find conf -not -name '*.cnf' -type f -delete
    rm -f $CSR_FILE $CHAIN_CRT
}

function prepare {
    clean
    echo 1000 > "$CONF_DIR/root_serial"
    cp /dev/null "$CONF_DIR/root_index.txt"
    echo 2000 > "$CONF_DIR/im_serial"
    cp /dev/null "$CONF_DIR/im_index.txt"
}

prepare
```

Все готово к созданию фальшивой, но технически корректной пары корневой сертификат / приватный ключ:

```bash
ROOT_URL="mycorp.com"
ROOT_ORG="MYCORP"
ROOT_CNF="$CONF_DIR/root_openssl.cnf"
ROOT_CRT="root.crt"
ROOT_KEY="root.key"
ROOT_EXP=5000
ROOT_SUB="/C=RU/ST=Moscow/L=Moscow/O=$ROOT_ORG/CN=$ROOT_URL"

# Generate root key
openssl genrsa -out $ROOT_KEY $KEY_BITS
# Generate root certificate
openssl req -config $ROOT_CNF -new -x509 -sha256 -extensions v3_ca \
    -key $ROOT_KEY -out $ROOT_CRT -days $ROOT_EXP -subj $ROOT_SUB
# Verify root certificate
openssl x509 -noout -text -in $ROOT_CRT
```

Хорошо, теперь создадим промежуточный сертификат с ключом:

```bash
IM_CRT="intermediate.crt"
IM_KEY="intermediate.key"
IM_CNF="$CONF_DIR/im_openssl.cnf"
IM_EXP=4000
IM_SUB="/C=RU/ST=Moscow/L=Moscow/O=${ROOT_ORG}/CN=department.${ROOT_URL}"

# Generate intermediate key
openssl genrsa -out $IM_KEY $KEY_BITS
# Generate intermediate request
openssl req -config $IM_CNF -new -key $IM_KEY -out $CSR_FILE -subj $IM_SUB
# Generate intermediate certificate
openssl ca \
    -config $ROOT_CNF -batch \
    -extensions v3_intermediate_ca -notext -md sha256 \
    -days $IM_EXP -in $CSR_FILE -out $IM_CRT
# Verify intermediate certificate
openssl x509 -noout -text -in $IM_CRT
openssl verify -CAfile $ROOT_CRT $IM_CRT
```

Обратите внимание, что команда на создание промежуточного сертификата берет конфигурацию
корневого (`$ROOT_CNF`), т.е. подписывать будем им. Также важно, чтобы срок подписываемого
был меньше срока подписывающего.

Ну а теперь, собственно, итог: 

```bash
SERVER_CRT="server.crt"
SERVER_KEY="server.key"
SERVER_EXP=365
SERVER_SUB="/C=RU/ST=Moscow/L=Moscow/O=MYSERVER/CN=myserver.com"

# Generate key
openssl genrsa -out $SERVER_KEY $KEY_BITS
# Generate request
openssl req \
    -config $IM_CNF -key $SERVER_KEY -new -sha256 \
    -out $CSR_FILE -subj $SERVER_SUB
# Generate certificate
openssl ca \
    -batch \
    -config $IM_CNF -days $SERVER_EXP \
    -extensions server_cert \
    -notext -md sha256 \
    -in $CSR_FILE -out $SERVER_CRT
# Verify certificate
cat $IM_CRT $ROOT_CRT > $CHAIN_CRT
openssl verify -CAfile $CHAIN_CRT $SERVER_CRT
```

Обратите внимение, что проверка осуществляется с указанием не только промежуточного,
но всех CA-сертификатов, предварительно собранных в один файл (`$CHAIN_CRT`).
И файл запроса, и сертификат создаются с использованием конфигурации промежуточного сертификата.

Темным пятном тут осталось содержимое файлов конфигурации. Они очень похожи на стандартные конфигурации 
`openssl`, за исключением нескольких параметров, касающихся путей размещения служебных файлов и сертификатов для подписи.
Вот эти параметры:

```bash
[ CA_default ]
dir               = conf
certs             = $dir
crl_dir           = $dir
new_certs_dir     = $dir
```

Для корневого:

```bash
database          = $dir/root_index.txt
serial            = $dir/root_serial
private_key       = $dir/../root.key
certificate       = $dir/../root.crt
```

Для промежуточного:

```bash
database          = $dir/im_index.txt
serial            = $dir/im_serial
private_key       = $dir/../intermediate.key
certificate       = $dir/../intermediate.crt
```

Строго говоря, проверка корректности уже была произведена в процессе создания.
Но так как мне нужно было делать это в Python, я использовал
библиотеку [pyOpenSSL](https://pyopenssl.org/en/stable/index.html), а именно
модуль [crypto](https://pyopenssl.org/en/stable/api/crypto.html). Для примера покажу, как это можно сделать без затей:

```python
from OpenSSL import crypto

# Prepare X509 objects
root_cert = crypto.load_certificate(
    crypto.FILETYPE_PEM, open('root.crt').read()
)
intermediate_cert = crypto.load_certificate(
    crypto.FILETYPE_PEM, open('intermediate.crt').read()
)
server_cert = crypto.load_certificate(
    crypto.FILETYPE_PEM, open('server.crt').read()
)

# Prepare X509 store
store = crypto.X509Store()
store.add_cert(root_cert)
store.add_cert(intermediate_cert)

# Verify
crypto.X509StoreContext(store, server_cert).verify_certificate()
```

Для `X509Store` можно добавить
[флаги проверок](https://pyopenssl.org/en/stable/api/crypto.html#OpenSSL.crypto.X509StoreFlags), 
которые все сломают. К примеру, если добавить вот такой флаг, то получим исключение
`OpenSSL.crypto.X509StoreContextError`:

```python
store.set_flags(crypto.X509StoreFlags.CRL_CHECK)
```

А все потому, что будет предпринята попытка скачать CRL-файл, указанный в дополнениях сертификата,
и эта попытка провалится. Как с этим бороться, как добавить `Issuer URL` и т.п. - это совсем другая история,
которая в принципе вся решается через конфигурацию `openssl`.

Весь пример доступен на [github](https://github.com/dstarod/fake-ssl-cert)
