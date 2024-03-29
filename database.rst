.. currentmodule:: flask

Деректер базасын анықтау және оған қол жеткізу
==============================

Қолданба пайдаланушылар мен жазбаларды сақтау үшін `SQLite`_ дерекқорын пайдаланады. Python :mod:`sqlite3`модулінде кірістірілген SQLite қолдауымен бірге келеді.

SQLite ыңғайлы, себебі ол жеке дерекқор серверін конфигурациялауды қажет етпейді және Python-ға ендірілген. Алайда, егер параллель сұраулар дерекқорға бір уақытта жазуға тырысса, олар баяулайды, өйткені әр жазба дәйекті түрде жүреді. Шағын қолданбалар мұны байқамайды. Үлкен болғаннан кейін сіз басқа дерекқорға ауысқыңыз келуі мүмкін.

Нұсқаулықта SQL туралы егжей-тегжейлі айтылмайды. Егер сіз онымен таныс болмасаңыз, SQLite құжаттары `language`_ сипаттайды.

.. _SQLite: https://sqlite.org/about.html
.. _language: https://sqlite.org/lang.html


Дерекқорға қосылыңыз
-----------------------

SQLite дерекқорымен (және басқа Python дерекқор кітапханаларының көпшілігімен) жұмыс істеу кезінде бірінші кезекте оған қосылым жасау керек. Кез келген сұраулар мен операциялар жұмыс аяқталғаннан кейін жабылатын қосылым арқылы орындалады.

Веб-қосымшаларда бұл байланыс әдетте сұранысқа байланысты болады. Ол сұрауды өңдеу кезінде бір сәтте жасалады және жауап жібермес бұрын жабылады.

.. code-block:: python
    :caption: ``flaskr/db.py``

    import sqlite3

    import click
    from flask import current_app, g


    def get_db():
        if 'db' not in g:
            g.db = sqlite3.connect(
                current_app.config['DATABASE'],
                detect_types=sqlite3.PARSE_DECLTYPES
            )
            g.db.row_factory = sqlite3.Row

        return g.db


    def close_db(e=None):
        db = g.pop('db', None)

        if db is not None:
            db.close()

:data:`g` - бұл әр сұранысқа ғана тән арнайы объект. Ол сұрау кезінде бірнеше мүмкіндіктерге қол жеткізуге болатын деректерді сақтау үшін қолданылады. Егер ``get_db`` сол сұрауда екінші рет шақырылса, қосылым сақталады және жаңа қосылым жасаудың орнына қайта пайдаланылады.

:data:`current_app` - бұл сұранысты өңдейтін flask қосымшасын көрсететін тағы бір арнайы объект. Қолданбалар фабрикасын пайдаланғандықтан, кодтың қалған бөлігін жазу кезінде қолданба нысаны жоқ. ``get_db`` қолданба жасалып, сұранысты өңдеген кезде шақырылады, сондықтан :data:`current_app` қолдануға болады.

:func:`sqlite3.connect`-  ``DATABASE`` конфигурация кілті көрсетілген файлмен байланыс орнатады. Бұл файл әлі болмауы керек және кейінірек дерекқорды инициализацияламайынша болмайды.

:class:`sqlite3.Row` қосылымды сөздік сияқты әрекет ететін жолдарды қайтаруға нұсқайды. Бұл бағандарға аты бойынша қол жеткізуге мүмкіндік береді.

``close_db`` байланыстың жасалғанын тексереді, ``g.db``орнатылғанын тексереді. Егер Байланыс болса, ол жабылады. Әрі қарай, сіз өзіңіздің қосымшаңызға қосымшалар фабрикасындағы ``close_db`` функциясы туралы айтып бересіз, сонда ол әр сұраудан кейін шақырылады.


Кестелер жасау
-----------------

SQLite-де деректер *tables*  және *columns* сақталады. Олар деректерді сақтауға және алуға дейін жасалуы керек. Flaskr пайдаланушыларды ``user``  кестесінде, ал жазбаларды ``post`` кестесінде сақтайды. Бос кестелер жасау үшін қажет SQL командалары бар файл жасаңыз:

.. code-block:: sql
    :caption: ``flaskr/schema.sql``

    DROP TABLE IF EXISTS user;
    DROP TABLE IF EXISTS post;

    CREATE TABLE user (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      username TEXT UNIQUE NOT NULL,
      password TEXT NOT NULL
    );

    CREATE TABLE post (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      author_id INTEGER NOT NULL,
      created TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
      title TEXT NOT NULL,
      body TEXT NOT NULL,
      FOREIGN KEY (author_id) REFERENCES user (id)
    );

Осы SQL командаларын іске қосатын Python мүмкіндіктерін қосыңыз ``db.py``  файл:

.. code-block:: python
    :caption: ``flaskr/db.py``

    def init_db():
        db = get_db()

        with current_app.open_resource('schema.sql') as f:
            db.executescript(f.read().decode('utf8'))


    @click.command('init-db')
    def init_db_command():
        """Clear the existing data and create new tables."""
        init_db()
        click.echo('Initialized the database.')

:meth:`open_resource() <Flask.open_resource>`  - ``flask``бумасына қатысты файлды ашады, бұл пайдалы, өйткені қолданбаны кейінірек орналастырған кезде бұл орынның қай жерде екенін білудің қажеті жоқ. ``get_db`` файлдан оқылған командаларды орындау үшін пайдаланылатын мәліметтер базасына қосылымды қайтарады.

:func:`click.command` - ``init-db`` функциясын шақыратын және пайдаланушыға сәтті аяқталғаны туралы хабарлама көрсететін ``init-db``деп аталатын пәрмен жолын анықтайды. Пәрмендерді жазу туралы көбірек білу үшін :doc:`/cli` оқи аласыз.


Қолданбаға тіркеліңіз
-----------------------------

``close_db``  және ``init_db_command``  функциялары қолданба данасында тіркелуі керек; әйтпесе оларды қолданба пайдаланбайды. Алайда, сіз зауыттық функцияны қолданғандықтан, бұл функция функцияларды жазу кезінде қол жетімді емес. Оның орнына қосымшаны қабылдайтын және тіркеуді орындайтын функцияны жазыңыз.

.. code-block:: python
    :caption: ``flaskr/db.py``

    def init_app(app):
        app.teardown_appcontext(close_db)
        app.cli.add_command(init_db_command)

:meth:`app.teardown_appcontext() <Flask.teardown_appcontext>` жауап қайтарылғаннан кейін тазалау кезінде Flask функциясын шақыруға нұсқау береді.

:meth:`app.cli.add_command() <click.Group.add_command>`  flask пәрменімен шақыруға болатын жаңа пәрменді қосады.

Бұл мүмкіндікті Өндіруші зауыттан импорттаңыз және шақырыңыз. Қолданбаны қайтармас бұрын зауыттық мүмкіндіктің соңына жаңа кодты қойыңыз.

.. code-block:: python
    :caption: ``flaskr/__init__.py``

    def create_app():
        app = ...
        # existing code omitted

        from . import db
        db.init_app(app)

        return app


Дерекқор файлын инициализациялаңыз
----------------------------

.. note::

   Егер сіз әлі де алдыңғы беттен серверді іске қосып жатсаңыз, серверді тоқтатуға немесе бұл пәрменді жаңа терминалда іске қосуға болады. Егер сіз жаңа терминалды қолдансаңыз, жоба каталогына өтіп, сипатталғандай env-ді іске қосыңыз :doc:`/installation`.

``init-db``пәрмені қолдаңыз:

.. code-block:: none

    $ flask --app flaskr init-db
    Initialized the database.

Енді сіздің жобаңыздың``instance`` қалтасында``flask.sqlite`` файлы болады.

:doc:`views`- ге өтіңіз.
