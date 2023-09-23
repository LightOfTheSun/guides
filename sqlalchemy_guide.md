# SQLALCHEMY

<!-- TOC -->
* [SQLALCHEMY](#sqlalchemy)
  * [Базові функції](#базові-функції)
    * [<a id="engine"></a> create_engine()](#a-idenginea-createengine)
    * [declarative_base()](#declarativebase)
    * [MetaData()](#metadata)
    * [metadata.create_all()](#metadatacreateall)
    * [Session() <a id="session"></a>](#session-a-idsessiona)
    * [sessionmaker()](#sessionmaker)
  * [<a id="models"></a> Моделі `sqlalchemy` (опис таблиць у вигляді класів)](#a-idmodelsa-моделі-sqlalchemy-опис-таблиць-у-вигляді-класів)
    * [Column()](#column)
    * [ForeignKey()](#foreignkey)
    * [relationship()](#relationship)
      * [додаткові аргументи relationship():](#додаткові-аргументи-relationship)
      * [використання `back_populates`:](#використання-backpopulates-)
  * [Запити до бази даних](#запити-до-бази-даних)
    * [session.query()](#sessionquery)
    * [.first()](#first)
    * [.all()](#all)
    * [.filter() або .where()](#filter-або-where)
      * [Простий приклад використання `.filter()`:](#простий-приклад-використання-filter)
      * [Фільтрація з кількома умовами:](#фільтрація-з-кількома-умовами)
      * [Оператори логічного вибору:](#оператори-логічного-вибору)
      * [Інші корисні оператори фільтрації:](#інші-корисні-оператори-фільтрації)
        * [`.in_()`](#in)
        * [`.like()`](#like)
        * [`.is_()`](#is)
        * [`.between()`](#between)
      * [Фільтрація за допомогою `relationship()` (метод `.has()`):](#фільтрація-за-допомогою-relationship-метод-has)
      * [Фільтрація за допомогою `relationship()` (метод `.any()`):](#фільтрація-за-допомогою-relationship-метод-any)
    * [`.order_by()`](#orderby)
    * [`.distinct()`](#distinct)
    * [`.group_by()`](#groupby)
    * [`.label()`](#label)
    * [`.limit()`](#limit)
    * [`.offset()`](#offset)
  * [Джойни (joins) таблиць](#джойни-joins-таблиць)
    * [INNER JOIN `.join()`](#inner-join-join)
    * [LEFT JOIN `.outerjoin()`](#left-join-outerjoin)
    * [`.union()` та `.union_all()`](#union-та-unionall)
    * [RIGHT JOIN](#right-join)
    * [FULL OUTER JOIN](#full-outer-join)
  * [Функції SQL](#функції-sql)
    * [Агрегатні функції](#агрегатні-функції)
      * [`.scalar()`](#scalar)
      * [Найпоширеніші агрегатні функції:](#найпоширеніші-агрегатні-функції)
    * [Інші функції SQL з модуля `func`](#інші-функції-sql-з-модуля-func)
      * [Текстові функції](#текстові-функції)
      * [Математичні функції](#математичні-функції)
      * [Дати та час](#дати-та-час)
      * [Логічні функції](#логічні-функції)
    * [`cast()`](#cast)
    * [`case()`](#case)
    * [Конструкція `row_number()` + `over()`](#конструкція-rownumber--over)
  * [Підзапити (subqueries та CTE)](#підзапити-subqueries-та-cte)
    * [`.subquery()`](#subquery)
    * [`.cte()`](#cte)
  * [INSERT](#insert)
    * [`.add()`](#add)
    * [`.add_all()`](#addall)
    * [`.insert()`](#insert-1)
    * [`.insert().returning()`](#insertreturning)
  * [UPDATE](#update)
    * [`.update()`](#update-1)
    * [`.update().returning()`](#updatereturning)
    * [`UPDATE` з використанням об'єктів, що були раніше вибрані з бази даних](#update-з-використанням-обєктів-що-були-раніше-вибрані-з-бази-даних)
  * [DELETE](#delete)
    * [`.delete()`](#delete-1)
    * [`DELETE` з використанням об'єктів, що були раніше вибрані з бази даних](#delete-з-використанням-обєктів-що-були-раніше-вибрані-з-бази-даних)
* [Додатковий словник термінів](#додатковий-словник-термінів)
<!-- TOC -->

## Базові функції

### <a id="engine"></a> create_engine()
створює об'єкт Engine, який відповідає за з'єднання з базою даних. 
Приймає рядок підключення до бази даних.

Приклад підключення до бази даних `MySQL`:
```python
from sqlalchemy import create_engine

SQLALCHEMY_DATABASE_URL = "mysql+pymysql://root:root@localhost:3306/test"
engine = create_engine(SQLALCHEMY_DATABASE_URL)
```

***

### declarative_base()
створює базовий клас для всіх класів моделей. Від нього будуть наслідуватися всі класи моделей.
Цей клас дозволяє нам описувати таблиці у вигляді класів, а стовпці у вигляді атрибутів цих класів.

```python
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
```

***

### MetaData()
створює об'єкт MetaData, який відповідає за збереження метаданих про таблиці.
Метадані - це інформація про таблиці, така як назва таблиці, назви стовпців, типи даних тощо.

```python
from sqlalchemy import MetaData

metadata = MetaData()
```

З об'єкту metadata можна отримати інформацію про таблиці, які він містить:
```python
print(metadata.tables)
```

***

### metadata.create_all()
створює всі таблиці, які описані у класах моделей.

***

### Session() <a id="session"></a>
створює об'єкт Session, який відповідає за сесії з базою даних.
Сесія - це тимчасове з'єднання з базою даних, яке використовується для виконання запитів до бази даних.

```python
from sqlalchemy.orm import Session

session = Session(bind=engine)
```

*bind* - вказує на те, який об'єкт [Engine](#engine) буде використовуватися для з'єднання з базою даних.

***

### sessionmaker()
створює фабрику сесій, яка відповідає за створення сесій з базою даних.

*Фабрика* - це функція, яка створює об'єкти певного типу.

*Фабрика сесій* - це функція, яка створює об'єкти [Session](#session).

Приклад створення і використання фабрики сесій:
```python
from sqlalchemy.orm import sessionmaker

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

*Пояснення*:
- `autocommit=False` - вимикає автоматичний коміт транзакцій. Це означає, що транзакції будуть комітитися тільки тоді, коли ми викличемо метод `commit()` у сесії.
Коміт - це процес, який зберігає всі зміни до бази даних. Якщо виникне помилка, то транзакція буде відкочена.
- `autoflush=False` - вимикає автоматичний "флаш" транзакцій. Це означає, що транзакції будуть "флашитися" тільки тоді, коли ми викличемо метод `flush()` у сесії.
Флаш - це процес, який відправляє всі накопичені зміни до бази даних, але не комітить транзакцію (тобто якщо виникне помилка, то транзакція буде відкочена).
- `bind=engine` - вказує, що сесії, які створює фабрика сесій, будуть прив'язані до об'єкту `engine`.
- `yield db` - вказує, що функція `get_db()` є генератором. Це означає, що вона буде повертати сесію, але не буде закривати її. Сесія буде закрита тільки тоді, коли викличеться метод `close()` у сесії.
- `db.close()` - закриває сесію.
- `finally` - код, який знаходиться після finally, буде виконуватися незалежно від того, чи виникла помилка у блоці try, чи ні.
- Приклад використання функції `get_db()`:
```python
def get_user(user_id: int):
    with get_db() as db:
        return db.query(User).filter(User.id == user_id).first()
```

db - це [сесія](#session), яку "їлдить" (`yield`) функція `get_db()`. За допомогою методу `query()` ми можемо виконувати запити до бази даних.

***

## <a id="models"></a> Моделі `sqlalchemy` (опис таблиць у вигляді класів)

*Приклад опису таблиці у вигляді класу:*
```python
from sqlalchemy import Column, Integer, String

class User(Base):
    __tablename__ = "users"
    __table_args__ = {'schema': 'dbo'}

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(50))
    email = Column(String(100), unique=True)
    password = Column(String(30))
```

*Пояснення:*
- **__tablename__** - назва таблиці у базі даних.
- **__table_args__** - додаткові аргументи для таблиці. В наведеному випадку вказуємо схему, в якій знаходиться таблиця.
- **id** - стовпець id типу `Integer`, який є первинним ключем (primary key) таблиці. Також цей стовпець є індексом (index).
Індекс - це структура даних, яка дозволяє швидко знаходити записи у таблиці за певними значеннями стовпців. 
Значення стовпців, за якими буде шукатися запис (індекси), повинні бути унікальними.
- **name** - стовпець name типу `String`, довжина якого не перевищує 50 символів. Аналог `VARCHAR(50)` у SQL.
- **email** - стовпець email типу `String`, довжина якого не перевищує 100 символів. Цей стовпець містить лише унікальні значення (unique=True).
- **password** - стовпець password типу `String`, довжина якого не перевищує 30 символів.

***

### Column()
створює стовпець у таблиці. Приймає тип даних стовпця та додаткові аргументи, серед яких можуть бути:
- **primary_key=True** - вказує, що цей стовпець є первинним ключем таблиці.
- **index=True** - вказує, що цей стовпець є індексом таблиці.
- **unique=True** - вказує, що цей стовпець містить лише унікальні значення.
- **nullable=False** - вказує, що цей стовпець не може містити значення NULL.
- **autoincrement=True** - вказує, що значення цього стовпця будуть автоматично збільшуватися при додаванні нового запису до таблиці.
- **server_default** - вказує значення за замовчуванням для цього стовпця, яке буде використовуватися на рівні бази даних при додаванні нового запису до таблиці.

Приклад використання додаткових аргументів:
```python
from sqlalchemy import Column, Integer, String

class User(Base):
    __tablename__ = "users"
    __table_args__ = {'schema': 'dbo'}

    id = Column(Integer, primary_key=True, index=True, autoincrement=True)
    name = Column(String(50), nullable=False)
    email = Column(String(100), unique=True)
    password = Column(String(30), server_default="123456")
```

***

### ForeignKey()
створює зовнішній ключ (foreign key) для стовпця. Приймає назву таблиці та стовпця, на який буде посилатися зовнішній ключ.

Приклад використання `ForeignKey()`:
```python
from sqlalchemy import Column, Integer, String, ForeignKey

class UserInfo(Base):
    __tablename__ = "user_info"
    __table_args__ = {'schema': 'dbo'}

    id = Column(Integer, primary_key=True, index=True, autoincrement=True)
    user_id = Column(Integer, ForeignKey("users.id"))   # посилання на стовпець id таблиці users
    address = Column(String(100))
```

>Також, замість назви таблиці та стовпця, на який буде посилатися зовнішній ключ, можна використовувати поле класу моделі, 
на яке буде посилатися зовнішній ключ (для цього потрібно імпортувати клас моделі, на який буде посилатися зовнішній ключ).
В цьому випадку ми зменшуємо ймовірність помилки у тексті, але можемо стикнутися з помилкою "circular dependency" (циклічна залежність).

**Циклічна залежність (circular dependency)** виникає тоді, коли файли імпортують один одного.
Наприклад, файл `user.py` імпортує файл `user_info.py`, а файл `user_info.py` імпортує файл `user.py`.
```python
from sqlalchemy import Column, Integer, String, ForeignKey
from user import User

class UserInfo(Base):
    __tablename__ = "user_info"
    __table_args__ = {'schema': 'dbo'}

    id = Column(Integer, primary_key=True, index=True, autoincrement=True)
    user_id = Column(Integer, ForeignKey(User.id))   # посилання на стовпець id таблиці users
    address = Column(String(100))
```

***

### relationship()
створює зв'язок між двома класами моделей. Приймає клас моделі, з якою буде створюватися зв'язок.

Приклад використання relationship():
```python
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship
from user import User

class UserInfo(Base):
    __tablename__ = "user_info"
    __table_args__ = {'schema': 'dbo'}

    id = Column(Integer, primary_key=True, index=True, autoincrement=True)
    user_id = Column(Integer, ForeignKey(User.id))
    address = Column(String(100))

    user = relationship(User, lazy="selectin")
```

> Зв'язок між класами моделей дозволяє нам отримувати об'єкти класів моделей, які пов'язані з поточним об'єктом.
Наприклад, якщо ми отримали об'єкт класу UserInfo, то ми можемо отримати об'єкт класу `User`, який пов'язаний з цим об'єктом.
```python
user_info: UserInfo = db.query(UserInfo).first()
user: User = user_info.user
print(user.name)
```

#### додаткові аргументи relationship():
- **back_populates** - вказує назву поля, яке пов'язане з поточним об'єктом. Це поле повинно бути описане у класі моделі, з якою створюється зв'язок.
- **lazy** - вказує, яким чином буде виконуватися запит до бази даних для отримання об'єкта, який пов'язаний з поточним об'єктом.
    - **subquery** - запит до бази даних буде виконуватися у вигляді підзапиту (subquery). 
    Це означає, що спочатку буде виконаний запит до бази даних для отримання об'єктів поточного класу моделі, 
    а потім буде виконаний запит до бази даних для отримання об'єктів класу моделі, з якою створюється зв'язок.
    - **joined** - запит до бази даних буде виконуватися у вигляді join.
    Це означає, що буде виконаний запит до бази даних, який об'єднає таблиці поточного класу моделі та класу моделі, з якою створюється зв'язок.
    - **selectin** - запит до бази даних буде виконуватися у вигляді selectin, що означає, що об'єкти з relationship будуть 
    вибиратися за допомогою додаткового запиту `WHERE id IN (SELECT ...)`. В більшості випадків цей тип запиту до бази даних є найшвидшим.
    - **noload** - запит до бази даних не буде виконуватися. Це означає, що об'єкт, який пов'язаний з поточним об'єктом, буде `None`.

***

#### використання `back_populates`: 
> Якщо використовувати `back_populates`, то потрібно вказати його у обох класах моделей, які пов'язані між собою.
При цьому, класи моделей у зв'язках повинні бути прописані через `str`, а не через клас:
```python
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship

class User(Base):
    __tablename__ = "users"
    __table_args__ = {'schema': 'dbo'}

    id = Column(Integer, primary_key=True, index=True, autoincrement=True)
    name = Column(String(50))
    email = Column(String(100), unique=True)
    password = Column(String(30))

    user_info = relationship("UserInfo", back_populates="user", lazy="selectin")
    
    
class UserInfo(Base):
    __tablename__ = "user_info"
    __table_args__ = {'schema': 'dbo'}

    id = Column(Integer, primary_key=True, index=True, autoincrement=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    address = Column(String(100))

    user = relationship("User", back_populates="user_info", lazy="selectin")
```

Але в цьому випадку ми зможемо отримати об'єкт класу `User`, який пов'язаний з об'єктом класу `UserInfo`:
```python
user_info: UserInfo = db.query(UserInfo).first()
user: User = user_info.user
print(user.name)
```
Або навпаки, об'єкт класу `UserInfo`, який пов'язаний з об'єктом класу `User`:
```python
user: User = db.query(User).first()
user_info: UserInfo = user.user_info
print(user_info.address)
```

***

## Запити до бази даних

### session.query()
створює запит до бази даних. Приймає клас моделі, з якої будуть вибиратися об'єкти, або список класів моделей, з яких будуть вибиратися об'єкти,
або список полів класів моделей, з яких будуть вибиратися об'єкти.

1. Приклад використання `.query()` з класом моделі:
    ```python
    with get_db() as session:
        user: User | None = session.query(User).first()
    ```
    
    Аналог SQL-запиту:
    ```mysql
    SELECT * FROM users LIMIT 1;
    ```

2. Приклад використання `.query()` зі списком класів моделей:
    ```python
    with get_db() as session:
        user: tuple[User, UserInfo] | None = session.query(User, UserInfo).first()
    ```
    
    Аналог SQL-запиту:
    ```mysql
    SELECT * FROM users, user_info LIMIT 1;
    ```
   
3. Приклад використання `.query()` зі списком полів класів моделей:
    ```python
    with get_db() as session:
        user: tuple[str, str] | None = session.query(User.name, UserInfo.address).first()
    ```
   
    Аналог SQL-запиту:
    ```mysql
    SELECT name, address FROM users, user_info LIMIT 1;
    ```
   
> Якщо використовувати `session.query()` зі списком класів моделей або зі списком полів класів моделей, то результатом буде кортеж,
який містить об'єкти класів моделей або значення полів класів моделей:
```python
with get_db() as session:
    user: tuple[User, UserInfo] | None = session.query(User, UserInfo).first()
    print(user)   # (<User>, <UserInfo>)
```

***

### .first()
виконує запит до бази даних та повертає перший об'єкт, який відповідає запиту.
Аналог SQL-запиту:
```mysql
SELECT * FROM users LIMIT 1;
```

* При використанні `query()` з класом моделі повертає об'єкт класу моделі або `None`, якщо об'єкт не знайдено.
* При використанні `query()` зі списком класів моделей повертає кортеж, який містить об'єкти класів моделей або `None`, якщо об'єкт не знайдено.
* При використанні `query()` зі списком полів класів моделей повертає кортеж, який містить значення полів класів моделей або `None`, якщо об'єкт не знайдено.

***

### .all()
виконує запит до бази даних та повертає всі об'єкти, які відповідають запиту.
Аналог SQL-запиту:
```mysql
SELECT * FROM users;
```

* При використанні `query()` з класом моделі повертає список об'єктів класу моделі (або пустий список, якщо об'єкти не знайдено).
```python
with get_db() as session:
    users: list[User] = session.query(User).all()
```
* При використанні `query()` зі списком класів моделей повертає список кортежів, які містять об'єкти класів моделей (або пустий список, якщо об'єкти не знайдено).
```python
with get_db() as session:
    users: list[tuple[User, UserInfo]] = session.query(User, UserInfo).all()
```
* При використанні `query()` зі списком полів класів моделей повертає список кортежів, які містять значення полів класів моделей (або пустий список, якщо об'єкти не знайдено).
```python
with get_db() as session:
    users: list[tuple[str, str]] = session.query(User.name, UserInfo.address).all()
```

***

### .filter() або .where()
виконує фільтрацію об'єктів, які відповідають запиту. Приймає умову(-и) фільтрації.

#### Простий приклад використання `.filter()`:
```python
with get_db() as session:
    users: list[User] = session.query(User).filter(User.name == "John").all()
```
Аналог SQL-запиту:
```mysql
SELECT * FROM users WHERE name = 'John';
```

***

#### Фільтрація з кількома умовами:
```python
with get_db() as session:
    users: list[User] = session.query(User).filter(User.name == "John", User.email == "John@gmail.com").all()
```
> При фільтрації з кількома умовами, всі умови фільтрації будуть об'єднані за допомогою оператора `AND`.
Аналог SQL-запиту:
```mysql
SELECT * FROM users WHERE name = 'John' AND email = 'John@gmail.com';
```

***

#### Оператори логічного вибору:
```python
from sqlalchemy import or_, and_

with get_db() as session:
    users: list[User] = (
        session.query(User)
        .filter(
            or_(User.name == "John", 
                and_(User.name == "Bob", User.email != "John@gmail.com"))
        )
        .all()
    )
```

* `or_` - оператор `OR`. Всі умови фільтрації, які знаходяться всередині `or_`, будуть об'єднані за допомогою оператора `OR`.
* `and_` - оператор `AND`. Всі умови фільтрації, які знаходяться всередині `and_`, будуть об'єднані за допомогою оператора `AND`.

Аналог SQL-запиту:
```mysql
SELECT * FROM users WHERE name = 'John' OR (name = 'Bob' AND email <> 'John@gmail.com');
```
> З бази даних будуть вибрані користувачі з іменем "John" або з іменем "Bob", в яких email не дорівнює "John@gmail.com".

***

#### Інші корисні оператори фільтрації:

##### `.in_()`
оператор `IN`.
```python
with get_db() as session:
    users: list[User] = session.query(User).filter(User.name.in_(["John", "Bob"])).all()
```
Аналог SQL-запиту:
```mysql
SELECT * FROM users WHERE name IN ('John', 'Bob');
```

***

##### `.like()`
оператор `LIKE`.
```python
with get_db() as session:
    users: list[User] = session.query(User).filter(User.name.like("%John%")).all()
```
Аналог SQL-запиту:
```mysql
SELECT * FROM users WHERE name LIKE '%John%';
```

***

##### `.is_()`
оператор `IS`.
```python
with get_db() as session:
    users: list[User] = session.query(User).filter(User.name.is_(None)).all()
```
Аналог SQL-запиту:
```mysql
SELECT * FROM users WHERE name IS NULL;
```

***

> Всі перераховані оператори мають свої інверсні аналоги: `.notin_()`, `.notlike()`, `.isnot()`.

***

##### `.between()`
виконує фільтрацію об'єктів, які знаходяться в межах заданих початкового та кінцевого значень.

```python
with get_db() as session:
    users: list[User] = session.query(User).filter(User.id.between(1, 10)).all()
```

Аналог SQL-запиту:
```mysql
SELECT * FROM users WHERE id BETWEEN 1 AND 10;
```

***

#### Фільтрація за допомогою `relationship()` (метод `.has()`):
```python
with get_db() as session:
    users: list[User] = (
        session.query(User)
        .filter(User.user_info.has(UserInfo.address.is_(None)))
        .all()
    )
```
> З бази даних будуть вибрані користувачі, які мають інформацію про себе, в якій адреса дорівнює `NULL`.

Аналог SQL-запиту:
```mysql
SELECT users.* FROM users
INNER JOIN user_info ON users.id = user_info.user_id
WHERE user_info.address IS NULL;
```

***

#### Фільтрація за допомогою `relationship()` (метод `.any()`):
```python
with get_db() as session:
    users: list[User] = (
        session.query(User)
        .filter(User.user_info.any(UserInfo.address.is_(None)))
        .all()
    )
```
> З бази даних будуть вибрані користувачі, які мають інформацію про себе, в якій адреса дорівнює `NULL`, або не мають інформації про себе зовсім. 

Аналог SQL-запиту:
```mysql
SELECT users.* FROM users
LEFT JOIN user_info ON users.id = user_info.user_id
WHERE user_info.address IS NULL;
```

> Різниця між `.has()` та `.any()` в тому, що `.has()` використовує `INNER JOIN`, а `.any()` використовує `LEFT JOIN`.

***

### `.order_by()`
виконує сортування об'єктів за певними полями. Приймає поля, за якими буде виконуватися сортування.

* `.asc()` - сортування за зростанням (за замовчуванням, можна опустити (приклад в коментарі)).
* `.desc()` - сортування за спаданням.

```python
with get_db() as session:
    users: list[User] = (
        session.query(User)
        .order_by(User.name.asc(), User.id.desc())
        # .order_by(User.name, User.id.desc())
        .all()
    )
```

Аналог SQL-запиту:
```mysql
SELECT * FROM users ORDER BY name ASC, id DESC;
# SELECT * FROM users ORDER BY name, id DESC;
```

***

### `.distinct()`
виконує вибірку лише унікальних значень.

```python
with get_db() as session:
    users: list[str] = session.query(User.name).distinct().all()
```

Аналог SQL-запиту:
```mysql
SELECT DISTINCT name FROM users;
```

***

### `.group_by()`
виконує групування об'єктів за певними полями. Приймає поля, за якими буде виконуватися групування.

```python
from sqlalchemy import func

with get_db() as session:
    users: list[tuple[str, int]] = (
        session.query(User.name, func.count(User.id))
        .group_by(User.name)
        .all()
    )
```

Аналог SQL-запиту:
```mysql
SELECT name, COUNT(id) FROM users GROUP BY name;
```

***

### `.label()`
виконує перейменування поля. Приймає нову назву поля. Аналог оператора `AS` у SQL.

```python
from typing import Any

with get_db() as session:
    user: tuple[str, int] | Any = (
        session.query(User.name.label("user_name"), func.count(User.id).label("users_count"))
        .group_by(User.name)
        .first()
    )
    # дані можна отримати також за назвою поля:
    user_name: str = user.user_name
    users_count: int = user.users_count
```

Аналог SQL-запиту:
```mysql
SELECT name AS user_name, COUNT(id) AS users_count FROM users GROUP BY name LIMIT 1;
```

***

### `.limit()`
виконує обмеження кількості об'єктів, які будуть вибрані з бази даних. Приймає кількість об'єктів, які будуть вибрані з бази даних.

```python
with get_db() as session:
    users: list[User] = session.query(User).limit(10).all()
```

Аналог SQL-запиту:
```mysql
SELECT * FROM users LIMIT 10;
```

***

### `.offset()`
виконує зміщення вибірки об'єктів. Приймає кількість об'єктів, на яку буде зміщена вибірка об'єктів.
При використанні `.offset()` необхідно використовувати `.order_by()`, щоб визначити порядок об'єктів, які будуть вибрані з бази даних.
Також, при використанні `.offset()` необхідно використовувати `.limit()`, щоб визначити кількість об'єктів, які будуть вибрані з бази даних.

```python
with get_db() as session:
    users: list[User] = (
        session.query(User)
        .order_by(User.id)
        .offset(10)
        .limit(10)
        .all()
    )
```

> З бази даних будуть вибрані користувачі, починаючи з 11-го та закінчуючи 20-м.

Аналог SQL-запиту:
```mysql
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 10;
```

***

## Джойни (joins) таблиць

### INNER JOIN `.join()`
виконує з'єднання таблиць. Приймає клас моделі, з якою буде виконуватися з'єднання.
Також може приймати додаткову(-і) умову(-и) з'єднання.

Приклад простого використання `.join()`:
```python
with get_db() as session:
    users: list[tuple[User, UserInfo]] = (
        session.query(User, UserInfo)
        .join(UserInfo)
        .all()
    )
```

Аналог SQL-запиту:
```mysql
SELECT * FROM users
INNER JOIN user_info ON users.id = user_info.user_id;
```
> Нам не потрібно вказувати умову з'єднання, оскільки SQLAlchemy самостійно визначить, за якими полями буде виконуватися з'єднання
(якщо в таблицях прописаний зв'язок через ключі, які SQLAlchemy може використати для з'єднання).

Також можна явно вказати умову з'єднання та/або додати додаткові умови з'єднання:
```python
from sqlalchemy import and_

with get_db() as session:
    users: list[tuple[User, UserInfo]] = (
        session.query(User, UserInfo)
        .join(UserInfo, and_(User.id == UserInfo.user_id, UserInfo.address == "Kyiv"))
        .all()
    )
```

Аналог SQL-запиту:
```mysql
SELECT * FROM users
INNER JOIN user_info ON users.id = user_info.user_id AND user_info.address = 'Kyiv';
```

***

### LEFT JOIN `.outerjoin()`
виконує з'єднання таблиць. Аналог `.join()`, але виконує з'єднання типу `LEFT JOIN`.
    
```python
with get_db() as session:
    users: list[tuple[User, UserInfo]] = (
        session.query(User, UserInfo)
        .outerjoin(UserInfo)
        .all()
    )
```

Аналог SQL-запиту:
```mysql
SELECT * FROM users
LEFT JOIN user_info ON users.id = user_info.user_id;
```

***

### `.union()` та `.union_all()`
виконує об'єднання результатів двох запитів до бази даних. Приймає запит до бази даних, з яким буде виконуватися об'єднання.
`.union()` виконує об'єднання без повторень (в результаті будуть унікальні значення), а `.union_all()` виконує об'єднання з повтореннями.

```python
with get_db() as session:
    users: list[User] = (
        session.query(User)
        .filter(User.name == "John")
        .union(session.query(User).filter(User.name == "Bob"))
        .all()
    )
```
> Також можемо винести дві `.query()` в окремі змінні (для кращої читабельності коду):
```python
with get_db() as session:
    query1 = session.query(User).filter(User.name == "John")
    query2 = session.query(User).filter(User.name == "Bob")
    users: list[User] = query1.union_all(query2).all()
```

Аналог SQL-запиту:
```mysql
SELECT * FROM users WHERE name = 'John'
UNION # або UNION ALL
SELECT * FROM users WHERE name = 'Bob';
```

***

### RIGHT JOIN
В SQLAlchemy немає методу `.rightjoin()`, але його можна замінити на `.outerjoin()` з використанням `select_from()`
або на `.outerjoin()`, змінивши порядок таблиць.

```python
with get_db() as session:
    users: list[tuple[User, UserInfo]] = (
        session.query(User, UserInfo)
        .select_from(UserInfo)
        .outerjoin(User)
        .all()
    )
# або
with get_db() as session:
    users: list[tuple[User, UserInfo]] = (
        session.query(UserInfo, User)
        .outerjoin(User)
        .all()
    )
```

Аналог SQL-запиту:
```mysql
SELECT * FROM users
RIGHT JOIN user_info ON users.id = user_info.user_id;
```

***

### FULL OUTER JOIN
В SQLAlchemy немає методу `.fullouterjoin()`.
Для його симуляції використовується `.union()` з двома `.outerjoin()`.
(В MySQL також немає FULL OUTER JOIN, тому там також використовується `UNION` з `LEFT JOIN` та `RIGHT JOIN`).

```python
with get_db() as session:
    left_join_query = (
        session.query(User, UserInfo)
        .outerjoin(UserInfo)
    )
    right_join_query = (
        session.query(UserInfo, User)
        .outerjoin(User)
    )
    users: list[tuple[User, UserInfo]] = left_join_query.union(right_join_query).all()
```

Аналог SQL-запиту:
```mysql
SELECT * FROM users
LEFT JOIN user_info ON users.id = user_info.user_id
UNION
SELECT * FROM users
RIGHT JOIN user_info ON users.id = user_info.user_id;
```

> В `PostgreSQL` та `MS SQL Server` можна використовувати `FULL OUTER JOIN` напряму:
```postgresql
SELECT * FROM users
FULL OUTER JOIN user_info ON users.id = user_info.user_id;
```

***
***

## Функції SQL
`func` - це клас, який містить функції SQL. Ці функції можна використовувати у запитах до бази даних.

### Агрегатні функції

#### `.scalar()`
повертає перший стовпець першого рядка, який відповідає запиту. Зазвичай використовується з агрегатними функціями SQL, 
щоб повернути єдине значення.

```python
from sqlalchemy import func

with get_db() as session:
    users_count: int = session.query(func.count(User.id)).scalar()
```

***

#### Найпоширеніші агрегатні функції:

* `func.count()` - повертає кількість записів у таблиці.
* `func.sum()` - повертає суму значень стовпця.
* `func.avg()` - повертає середнє значення стовпця.
* `func.min()` - повертає мінімальне значення стовпця.
* `func.max()` - повертає максимальне значення стовпця.

***

### Інші функції SQL з модуля `func`

#### Текстові функції

* `func.lower()` - перетворює всі символи рядка у нижній регістр.
* `func.upper()` - перетворює всі символи рядка у верхній регістр.
* `func.trim()` - видаляє пробіли з початку та кінця рядка.
* `func.ltrim()` - видаляє пробіли з початку рядка.
* `func.rtrim()` - видаляє пробіли з кінця рядка.
* `func.length()` - повертає довжину рядка.

#### Математичні функції

* `func.abs()` - повертає абсолютне значення числа.
* `func.round()` - округлює число. Приймає другим аргументом кількість знаків після коми:
```python
from sqlalchemy import func

with get_db() as session:
    number: float = session.query(func.round(1.2345, 2)).scalar()
    print(number)   # 1.23
```
* `func.floor()` - округлює число до меншого цілого.
* `func.ceil()` - округлює число до більшого цілого.

#### Дати та час

* `func.now()` - повертає поточну дату та час.
* `func.current_date()` - повертає поточну дату.
* `func.current_time()` - повертає поточний час.
* `func.current_timestamp()` - повертає поточну дату та час.
* `func.date()` - повертає дату з дати та часу.
Приклад використання:
```python
from sqlalchemy import func
from datetime import datetime

with get_db() as session:
    date: datetime.date = session.query(func.date(func.now())).scalar()
    print(date)   # 2023-09-23 (приклад)
```
* `func.time()` - повертає час з дати та часу.
* `func.extract()` - повертає частину дати та часу. Приймає другим аргументом частину дати та часу, яку потрібно повернути:
```python
from sqlalchemy import func

with get_db() as session:
    year: int = session.query(func.extract("year", func.now())).scalar()
    print(year)   # 2023
```
* `func.datediff()` - повертає різницю між двома датами. Приймає другим аргументом дату, від якої потрібно відняти першу дату:
```python
from sqlalchemy import func

with get_db() as session:
    days: int = session.query(func.datediff("2023-09-23", "2023-09-10")).scalar()
    print(days)   # 13
```

#### Логічні функції

* `func.coalesce()` - повертає перше значення, яке не дорівнює `NULL`. Приймає довільну кількість аргументів:
```python
from sqlalchemy import func

with get_db() as session:
    name: str = session.query(func.coalesce(None, "John")).scalar()
    print(name)   # John
```
* `func.nullif()` - повертає `NULL`, якщо два аргументи дорівнюють один одному. Приймає два аргументи:
```python
from sqlalchemy import func

with get_db() as session:
    name: str = session.query(func.nullif("John", "John")).scalar()
    print(name)   # None
```
* `func.ifnull()` - повертає другий аргумент, якщо перший аргумент дорівнює `NULL`. Приймає два аргументи:
```python
from sqlalchemy import func

with get_db() as session:
    name: str = session.query(func.ifnull(None, "John")).scalar()
    print(name)   # John
```
* `func.if()` - повертає другий аргумент, якщо перший аргумент дорівнює `True`, або третій аргумент, якщо перший аргумент дорівнює `False`. Приймає три аргументи:
```python
from sqlalchemy import func

with get_db() as session:
    name: str = session.query(func.if_(1 == 1, "John", "Bob")).scalar()
    print(name)   # John
    name: str = session.query(func.if_(0 == 1, "John", "Bob")).scalar()
    print(name)   # Bob
```

***

> Також через `func` можна використовувати будь-які інші функції SQL, які не підтримуються SQLAlchemy напряму.

***

### `cast()`
перетворює значення одного типу на інший тип. Приймає два аргументи: значення та тип, в який потрібно перетворити значення.

```python
from sqlalchemy import cast, Integer

with get_db() as session:
    number: int = session.query(cast(1.2345, Integer)).scalar()
    print(number)   # 1
```

Аналог SQL-запиту:
```mysql
SELECT CAST(1.2345 AS INT);
```

***

### `case()`
виконує умовну логіку. Приймає довільну кількість аргументів, які повинні бути кортежами з двох елементів: умова та значення.
Також може приймати додатковий аргумент `else_`, який вказує значення, яке повернеться, якщо жодна з умов не виконається.

```python
from sqlalchemy import case

with get_db() as session:
    name: str = (
        session.query(
            case(
                [
                    (User.name == "John", "John"),
                    (User.name == "Bob", "Bob"),
                ],
                else_="Other",
            )
        )
        .filter(User.id == 1)
        .scalar()
    )
    print(name)   # John
```

Аналог SQL-запиту:
```mysql
SELECT CASE
    WHEN name = 'John' THEN 'John'
    WHEN name = 'Bob' THEN 'Bob'
    ELSE 'Other'
END
FROM users
WHERE id = 1;
```

***

### Конструкція `row_number()` + `over()`
виконує нумерацію рядків. Має наступну структурну форму:
```python
from sqlalchemy import func

with get_db() as session:
    users: list[tuple[int, str]] = (
        session.query(
            func.row_number().over(
                partition_by=User.name,
                order_by=User.id,
            ).label("row_number"),
            User.name,
        )
        .all()
    )
    print(users)   # [(1, 'John'), (2, 'John')]
```

Аналог SQL-запиту:
```mysql
SELECT
    ROW_NUMBER() OVER (PARTITION BY name ORDER BY id) AS `row_number`,
    name
FROM users;
```

> `.over()` приймає аргументи, які вказуються у форматі `partition_by=...` та `order_by=...`.
`partition_by` вказує поля, за якими буде виконуватися групування, а `order_by` вказує поля, за якими буде виконуватися сортування.

***

## Підзапити (subqueries та CTE)

### `.subquery()`
виконує підзапит. Приймає запит (`.query()`) до бази даних, з яким буде виконуватися підзапит.
Можна додати аліас підзапиту, передавши його як аргумент `name` (`.subquery(name="...")`).
Доступ до полів підзапиту здійснюється через атрибут `.c`.

```python
with get_db() as session:
    subquery = (
        session.query(User.id, UserInfo.address)
        .filter(User.name == "John")
        .subquery(name="subq")
    )
    users: list[tuple[User, str]] = (
        session.query(User, subquery.c.address.label("john_address"))
        .outerjoin(subquery, User.id == subquery.c.id)
        .all()
    )
```

Аналог SQL-запиту:
```mysql
SELECT users.*, subq.address AS john_address
FROM users
LEFT JOIN (
    SELECT id, address
    FROM user_info
    WHERE name = 'John'
) AS subq ON users.id = subq.id;
```

***

### `.cte()`
створює спільну таблицю виразів (common table expression). Приймає запит (`.query()`) до бази даних, з яким буде виконуватися підзапит.
Можна додати аліас підзапиту, передавши його як аргумент `name` (`.cte(name="...")`).
Доступ до полів підзапиту здійснюється через атрибут `.c`.

> CTE часто використовується для рекурсивних запитів. Рекурсивні запити використовуються для роботи з деревоподібними структурами даних.
Для прикладу, деревоподібна структура даних може використовуватися для зберігання ієрархії категорій товарів.

```python
from sqlalchemy import select, union_all, func

# Нехай в нас є таблиця категорій товарів, яка посилається сама на себе (для зберігання ієрархії категорій):
class Category(Base):
    __tablename__ = "categories"

    id = Column(Integer, primary_key=True)
    name = Column(String(255), nullable=False)
    parent_id = Column(Integer, ForeignKey("categories.id"))

    parent = relationship("Category", remote_side=[id], backref="children")
    # remote_side=[id] - вказує, що поле `parent_id` посилається на поле `id` цієї ж таблиці.
    # backref="children" - вказує, що відносини `parent` будуть зберігатися у полі `children`.

# Для того, щоб вибрати всі категорії, які є кореневими (тобто, які не мають батьківських категорій),
# ми можемо використати рекурсивний запит:

with get_db() as session:
    # Спочатку створюємо CTE, який вибирає всі категорії, які мають батьківські категорії:
    cte = (
        session.query(Category.id, Category.parent_id)
        .filter(Category.parent_id.isnot(None))
        .cte(name="cte", recursive=True)    # recursive=True - вказує, що цей CTE є рекурсивним
    )

    # Додаємо до CTE запит, який вибирає всі категорії, які мають батьківські категорії:
    cte = cte.union_all(
        session.query(Category.id, Category.parent_id)
        .filter(Category.parent_id.in_(cte.c.id))
    )

    # Виконуємо запит до бази даних, який вибирає всі категорії, які є кореневими:
    categories: list[Category] = (
        session.query(Category)
        .filter(Category.id.notin_(cte.select().with_only_columns([cte.c.id])))
        .all()
    )
    # .select() - виконує запит до бази даних, який вибирає всі категорії, які мають батьківські категорії.
    # .with_only_columns([cte.c.id]) - фільтрує результат запиту з CTE, залишаючи лише поле `id`.
```

Аналог SQL-запиту:
```mysql
WITH RECURSIVE cte AS (
    SELECT id, parent_id FROM categories WHERE parent_id IS NOT NULL
    UNION ALL
    SELECT id, parent_id FROM categories WHERE parent_id IN (SELECT id FROM cte)
)
SELECT * FROM categories WHERE id NOT IN (SELECT id FROM cte);
```

***

## INSERT

### `.add()`
додає об'єкт до бази даних. Приймає об'єкт, який буде доданий до бази даних.

```python
with get_db() as session:
    user = User(name="John")
    session.add(user)
    session.commit()
```

Аналог SQL-запиту:
```mysql
INSERT INTO users (name) VALUES ('John');
```

***

### `.add_all()`
додає декілька об'єктів до бази даних. Приймає список об'єктів, які будуть додані до бази даних.

```python
with get_db() as session:
    users = [
        User(name="John"),
        User(name="Bob"),
    ]
    session.add_all(users)
    session.commit()
```

Аналог SQL-запиту:
```mysql
INSERT INTO users (name) VALUES ('John'), ('Bob');
```

***

### `.insert()`
виконує запит до бази даних на додавання даних. Приймає словник з даними, які будуть додані до бази даних.

```python
from sqlalchemy import insert

with get_db() as session:
    query = insert(User).values(name="John")
    session.execute(query)
    session.commit()
```

Аналог SQL-запиту:
```mysql
INSERT INTO users (name) VALUES ('John');
```

***

### `.insert().returning()`
виконує запит до бази даних на додавання даних. Приймає словник з даними, які будуть додані до бази даних.
Також може повертати дані, які були додані до бази даних. Для цього використовується метод `.returning()`.

```python
from sqlalchemy import insert

with get_db() as session:
    query = insert(User).values(name="John").returning(User.id)
    user_id: int = session.execute(query).scalar()
    session.commit()
```

Аналог SQL-запиту:
```mysql
INSERT INTO users (name) VALUES ('John');
SELECT LAST_INSERT_ID();
```

***

## UPDATE

### `.update()`
виконує запит до бази даних на оновлення даних. Приймає словник з даними, які будуть оновлені в базі даних.

```python
from sqlalchemy import update

with get_db() as session:
    query = update(User).values(name="Bob").where(User.id == 1)
    session.execute(query)
    session.commit()
```

Аналог SQL-запиту:
```mysql
UPDATE users SET name = 'Bob' WHERE id = 1;
```

***

### `.update().returning()`
виконує запит до бази даних на оновлення даних. Приймає словник з даними, які будуть оновлені в базі даних.
Також може повертати дані, які були оновлені в базі даних. Для цього використовується метод `.returning()`.

```python
from sqlalchemy import update

with get_db() as session:
    query = update(User).values(name="Bob").where(User.id == 1).returning(User.name)
    user_name: str = session.execute(query).scalar()
    session.commit()
```

Аналог SQL-запиту:
```mysql
UPDATE users SET name = 'Bob' WHERE id = 1;
SELECT name FROM users WHERE id = 1;
```

***

### `UPDATE` з використанням об'єктів, що були раніше вибрані з бази даних

```python
with get_db() as session:
    # Вибираємо користувача з бази даних:
    user: User = session.query(User).filter(User.id == 1).first()
    # Змінюємо дані користувача:
    user.name = "Bob"
    # Зберігаємо зміни в базі даних:
    session.commit()
```

## DELETE

### `.delete()`
виконує запит до бази даних на видалення даних.

```python
with get_db() as session:
    query = delete(User).where(User.id == 1)
    session.execute(query)
    session.commit()
```

Аналог SQL-запиту:
```mysql
DELETE FROM users WHERE id = 1;
```

***

### `DELETE` з використанням об'єктів, що були раніше вибрані з бази даних

```python
with get_db() as session:
    # Вибираємо користувача з бази даних:
    user: User = session.query(User).filter(User.id == 1).first()
    # Видаляємо користувача з бази даних:
    session.delete(user)
    # Зберігаємо зміни в базі даних:
    session.commit()
```

***

***

***

# Додатковий словник термінів

* **ORM** - Object-Relational Mapping (об'єктно-реляційне відображення).
Це технологія, яка дозволяє використовувати об'єктноорієнтовані класи для роботи з 
реляційними базами даних.
* **SQL** - Structured Query Language (структурована мова запитів).
* **Commit** - фіксує зміни в базі даних. Після коміту зміни в базі даних стають постійними.
* **Rollback** - відміняє зміни в базі даних. Після ролбеку зміни в базі даних не зберігаються.
* **Session** - сесія. Це об'єкт, який використовується для взаємодії з базою даних. Сесія містить в собі транзакцію.
* **Transaction** - транзакція. Це блок коду, який виконується як єдиний цілий. Якщо транзакція успішно виконується, то вона комітиться.
* **Query** - запит до бази даних. Це об'єкт, який містить в собі запит до бази даних.
Якщо використати функцію `print()` для об'єкта запиту, то можна побачити SQL-запит, який буде виконуватися до бази даних:
```python
with get_db() as session:
    query = session.query(User)
    print(query)   # SELECT * FROM users
```
* **Table** - таблиця. Це клас, який відображає таблицю в базі даних.
* **Column** - стовпець таблиці. Це клас, який відображає стовпець таблиці в базі даних.
* **Relationship** - відносини між таблицями. Це клас, який відображає відносини між таблицями в базі даних.
* **ForeignKey** - зовнішній ключ. Це стовпець таблиці, який посилається на інший стовпець таблиці.
* **Primary Key** - первинний ключ. Це стовпець таблиці, який ідентифікує запис у таблиці.
* **Unique Key** - унікальний ключ. Це стовпець таблиці, який має унікальні значення.
* **Sessionmaker** - це клас, який створює сесію. Його можна використовувати для створення сесії.
* **Engine** - це клас, який використовується для підключення до бази даних.



