# NOTE (workshop на Flask)

## Действия в течении workshop

1. Создание папки;
   
```bash
mkdir workshop_1
``` 

2. Cоздание виртуального окружения; 
```python
python3 -m venv venv
```
    2.1. Запуск виртуального окружения;
```python
source venv/bin/activate
```
    2.2. Выход из виртуального окружения;
```python
deactivate
```
Источник: [руководство по созданию виртуального окружения](https://mob25.com/visual-studio-code-virtualnoe-okruzhenie-venv/); 

3. Cоздание репозитория в git hub;

[Источник по установки git](https://htmlacademy.ru/blog/git/git-console)

4. Создание requirements.txt;
```python
pip freeze > requirements.txt
```
5. Создание прогарммы.

### Пояснение

  Рассмотрим пункт 5 по лучше. Для более правильного написания програамы, воспользуемся подходом "чистая архитектура для веб-приложений".

- Описание файла model.py
```python
class Note:
    id: str
    title: str
    text: str
```
Данная модель ничего не знает о логиге, о требуемых параметрах. Данный объект всего лишь предоставляет данные для объмена между разными слоями.

- Описание файла server.py
```python
from api import app
app
```
Вызываем app для того что бы она была доступна из сервера. Потому что в настоящих и больших приложениях в этом можуле будут лежать настройки сервера.
- Описание api.py
```python
from flask import Flask
from flask import request

app = Flask(__name__)


import model
import logic

_note_logic = logic.NoteLogic()


class ApiException(Exception): #  Здесь нет логики. Только для разделения
    pass


def _from_raw(raw_note: str) -> model.Note: # Функция конвертации данных в модель (raw) - сырые дынные
    parts = raw_note.split('|')
    if len(parts) == 2: # Проверка по длине 'kolia|love'
        note = model.Note()
        note.id = None
        note.title = parts[0]
        note.text = parts[1]
        return note
    elif len(parts) == 3:# '1|kolia|love'
        note = model.Note()
        note.id = parts[0]
        note.title = parts[1]
        note.text = parts[2]
        return note
    else:
        raise ApiException(f"invalid RAW note data {raw_note}")


def _to_raw(note: model.Note) -> str:
    if note.id is None:
        return f"{note.title}|{note.text}"
    else:
        return f"{note.id}|{note.title}|{note.text}"


API_ROOT = "/api/v1" ## корень нашего URL
NOTE_API_ROOT = API_ROOT + "/note" ## корень нашего URL


@app.route(NOTE_API_ROOT + "/", methods=["POST"]) # Декоратор создания заметки
def create():
    try:
        data = request.get_data().decode('utf-8')#По умолчанию приходят данные в битовоц системе. decode('utf-8') декодируем данные
        note = _from_raw(data)
        _id = _note_logic.create(note)
        return f"new id: {_id}", 201
    except Exception as ex:
        return f"failed to CREATE with: {ex}", 404


@app.route(NOTE_API_ROOT + "/", methods=["GET"]) # Декоратор  заметки
def list():
    try:
        notes = _note_logic.list()
        raw_notes = ""
        for note in notes:
            raw_notes += _to_raw(note) + '\n'
        return raw_notes, 200
    except Exception as ex:
        return f"failed to LIST with: {ex}", 404


@app.route(NOTE_API_ROOT + "/<_id>/", methods=["GET"])# Декоратор чтения заметки
def read(_id: str):
    try:
        note = _note_logic.read(_id)
        raw_note = _to_raw(note)
        return raw_note, 200
    except Exception as ex:
        return f"failed to READ with: {ex}", 404


@app.route(NOTE_API_ROOT + "/<_id>/", methods=["PUT"])# Декоратор для изменения заметки
def update(_id: str):
    try:
        data = request.get_data().decode('utf-8')
        note = _from_raw(data)
        _note_logic.update(_id, note)
        return "updated", 200
    except Exception as ex:
        return f"failed to UPDATE with: {ex}", 404


@app.route(NOTE_API_ROOT + "/<_id>/", methods=["DELETE"])# Декоратор удаления заметки
def delete(_id: str):
    try:
        _note_logic.delete(_id)
        return "deleted", 200
    except Exception as ex:
        return f"failed to DELETE with: {ex}", 404
```




#### Литература:
> 1. [Чистая Архитектура для веб-приложений](https://habr.com/ru/articles/493430/) 
> 2. [curl — учимся тестировать API](https://testengineer.ru/curl-uchimsya-testirovat-api/) 
> 3. [Типы Python](https://www.youtube.com/watch?v=29WDYmT4e1E)




