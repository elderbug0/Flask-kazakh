Project Layout
==============

Жоба каталогын жасаңыз және оны енгізіңіз:

.. code-block:: none

    $ mkdir flask-tutorial
    $ cd flask-tutorial

Содан кейін Python виртуалды ортасын орнату және Flask орнату үшін :doc:`орнату нұсқауларын</installation>` орындаңыз. Жобаңыз үшін Python виртуалды ортасы және Flask орнату керек.

Оқулық сіз ``flask-tutorial`` каталогында жұмыс істеп жатырсыз деп болжайды.
каталогы ``flask-tutorial`` . Әрбір код блогының жоғарғы жағындағы файл атаулары сол каталогқа қатысты.
----

Flask қолданбасы бір файлдан тұруы мүмкін.

.. code-block:: python
    :caption: ``hello.py``

    from flask import Flask

    app = Flask(__name__)


    @app.route('/')
    def hello():
        return 'Hello, World!'

Дегенмен, жоба өсіп келе жатқанда, барлық кодты бір файлда сақтау қиын тапсырмаға айналады. Python жобалары кодты қажетінше импорттауға болатын бірнеше модульдерге ұйымдастыру үшін  *бумаларды*  пайдаланады және бұл оқулық мұны да жасайды.

Жоба каталогында мыналар болады:

* ``flaskr/``, қолданба коды мен файлдары бар Python бумасы.
   файлдар.
* ``tests/``, сынақ модульдері бар каталог.
* ``.venv/``, Flask және басқа тәуелділіктер орнатылған Python виртуалды ортасы.
   тәуелділіктер.
* Орнату файлдары Python-ға жобаңызды қалай орнату керектігін айтады.
* Нұсқаны басқару конфигурациясы, мысалы, ``git``. Сіз оны ережеге айналдыруыңыз керек
   өлшеміне қарамастан барлық жобаларыңыз үшін нұсқаны басқарудың кейбір түрін пайдаланыңыз.
   өлшемі.
* Болашақта қосуға болатын кез келген басқа жоба файлдары.

.. _git: https://git-scm.com/

Нәтижесінде жобаның диаграммасы келесідей болады:

.. code-block:: none

    /home/user/Projects/flask-tutorial
    ├── flaskr/
    │   ├── __init__.py
    │   ├── db.py
    │   ├── schema.sql
    │   ├── auth.py
    │   ├── blog.py
    │   ├── templates/
    │   │   ├── base.html
    │   │   ├── auth/
    │   │   │   ├── login.html
    │   │   │   └── register.html
    │   │   └── blog/
    │   │       ├── create.html
    │   │       ├── index.html
    │   │       └── update.html
    │   └── static/
    │       └── style.css
    ├── tests/
    │   ├── conftest.py
    │   ├── data.sql
    │   ├── test_factory.py
    │   ├── test_db.py
    │   ├── test_auth.py
    │   └── test_blog.py
    ├── .venv/
    ├── pyproject.toml
    └── MANIFEST.in

Нұсқаны басқаруды пайдаланып жатсаңыз, жоба іске қосылғанда жасалған келесі файлдар еленбеуі керек. Қолданылатын өңдегішке байланысты басқа файлдар болуы мүмкін. Жалпы, жазбаған файлдарды елемеңіз. Мысалы, git пайдаланған кезде:

.. code-block:: none
    :caption: ``.gitignore``

    .venv/

    *.pyc
    __pycache__/

    instance/

    .pytest_cache/
    .coverage
    htmlcov/

    dist/
    build/
    *.egg-info/

Жалғастыру :doc:`factory`.
