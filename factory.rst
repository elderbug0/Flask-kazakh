.. currentmodule:: flask

Қолданбаны орнату
=================

Flask қолданбасы :class:`Flask` сыныбының данасы болып табылады. Конфигурация және URL мекенжайлары сияқты қолданбаға қатысты барлық нәрсе осы сыныпта тіркеледі.

Flask қосымшасын құрудың ең оңай жолы- global :class:`Flask` данасын тікелей кодтың жоғарғы жағында жасау, мысалы, "Hello, World!" алдыңғы бетте. Бұл кейбір жағдайларда қарапайым және пайдалы болғанымен, жоба өскен сайын кейбір күрделі мәселелерді тудыруы мүмкін.

:class:`Flask` ғаламдық дананы құрудың орнына, сіз оны функцияның ішінде жасайсыз. Бұл функция *application factory* деп аталады. Қолданбаға қажет кез келген конфигурация, тіркеу және басқа параметрлер функция ішінде орындалады, содан кейін қолданба қайтарылады.

Қолданбалар фабрикасы
-----------------------

Кодтауды бастау уақыты келді!``flaskr`` каталогын жасаңыз және  ``__init__.py`` файл қосыңыз. Сол  ``__init__.py`` қос функцияны орындайды: онда қосымшалар фабрикасы болады және Python-ға ``flaskr``каталогы пакет ретінде қарастырылуы керек деп хабарлайды.

.. code-block:: none

    $ mkdir flaskr

.. code-block:: python
    :caption: ``flaskr/__init__.py``

    import os

    from flask import Flask


    def create_app(test_config=None):
        # create and configure the app
        app = Flask(__name__, instance_relative_config=True)
        app.config.from_mapping(
            SECRET_KEY='dev',
            DATABASE=os.path.join(app.instance_path, 'flaskr.sqlite'),
        )

        if test_config is None:
            # load the instance config, if it exists, when not testing
            app.config.from_pyfile('config.py', silent=True)
        else:
            # load the test config if passed in
            app.config.from_mapping(test_config)

        # ensure the instance folder exists
        try:
            os.makedirs(app.instance_path)
        except OSError:
            pass

        # a simple page that says hello
        @app.route('/hello')
        def hello():
            return 'Hello, World!'

        return app

``create_app`` is the application factory function. You'll add to it
later in the tutorial, but it already does a lot.

#.  ``app = Flask(__name__, instance_relative_config=True)`` :class`Flask` данасын жасайды.

    *   ``__name__`` бұл ағымдағы Python Модулінің атауы. Кейбір жолдарды конфигурациялау үшін қолданба қай жерде екенін білуі керек және ``__name__`` оған хабарлаудың ыңғайлы жолы болып табылады.

    *   ``instance_relative_config=True`` қолданбаға конфигурация файлдары :ref:`instance folder <instance-folders>` - ға байланасты екенін айтады. Даналық қалтасы ``flask`` бумасының сыртында орналасқан және конфигурация құпиялары мен дерекқор файлы сияқты нұсқаны басқару жүйесіне жіберілмейтін жергілікті деректерді қамтамайды.

#.  :meth:`app.config.from_mapping() <Config.from_mapping>` қолданба пайдаланатын кейбір әдепкі конфигурацияны орнатады:

    *   :data:`SECRET_KEY`деректер қауіпсіздігін қамтамасыз ету үшін Flask және кеңейтімдер қолданылады. Ол әзірлеу кезінде ыңғайлы мәнді қамтамасыз ету үшін ``'dev'`` мәніне орнатылған, бірақ орналастырылған кезде оны кездейсоқ мәнмен ауыстыру керек.

    *   ``DATABASE`` бұл SQLite дерекқор файлы сақталатын жол. Даналық қалтасы үшін Flask таңдаған жол бұл :attr:`app.instance_path <Flask.instance_path>`астында, Дерекқор туралы толығырақ келесі бөлімде білесіз.

#.  :meth:`app.config.from_pyfile() <Config.from_pyfile>` әдепкі конфигурацияны алынған мәндермен қайта анықтайды ``config.py``егер бар болса, даналық қалтасындағы файл. Мысалы, орналастыру кезінде оны нақты ``SECRET_KEY`` орнату үшін пайдалануға болады.

    *   ``test_config`` сонымен қатар зауытқа жіберілуі мүмкін және дананың конфигурациясының орнына қолданылады. Бұл нұсқаулықта кейінірек жазатын сынақтарды әзірлеу үшін Сіз орнатқан кез келген мәндерге қарамастан реттеуге болатындай етіп жасалады.

#.  :func:`os.makedirs` кепілдік береді :attr:`app.instance_path <Flask.instance_path>` бар. Flask даналық қалтаны автоматты түрде жасамайды, бірақ оны жасау керек, себебі сіздің жобаңыз сол жерде SQLite дерекқор файлын жасайды.

#.  :meth:`@app.route() <Flask.route>` нұсқаулықтың қалған бөлігіне өтпес бұрын қолданбаның қалай жұмыс істейтінін көру үшін қарапайым маршрут жасайды. Бұл ``/hello`` URL мекенжайы мен жауапты қайтаратын функция арасында байланыс жасайды, бұл жағдайда ``Сәлем Әлем!``.


Қолданбаны Іске Қосыңыз
-------------------

Енді қолданбаны ``flask`` пәрме арқылы іске қосуға болады. Терминалдан flask-ке қолданбаны қайдан табуға болатынын айтыңыз, содан кейін оны жөндеу режимінде іске қосыңыз. Есіңізде болсын, сіз әлі де``flaskr``  пакетінде емес, ``flask`` жоғарғы деңгейлі каталогында болуыңыз керек.

Отладка режимі интерактивті отладчикті бетте ерекше жағдайлар шыққан кезде көрсетеді және кодқа өзгертулер енгізген сайын серверді қайта іске қосады. Сіз оны қосулы күйде қалдыра аласыз және нұсқауларды орындау арқылы шолғыш бетін қайта жүктей аласыз.

.. code-block:: text

    $ flask --app flaskr run --debug

You'll see output similar to this:

.. code-block:: text

     * Serving Flask app "flaskr"
     * Debug mode: on
     * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
     * Restarting with stat
     * Debugger is active!
     * Debugger PIN: nnn-nnn-nnn

Сапар http://127.0.0.1:5000/hello браузерде және сіз "Hello, World!". Құттықтаймыз, енді сіз flask веб-қосымшасын іске қостыңыз!

Егер басқа бағдарлама 5000 портын қолданып жатса, сіз серверді іске қосқан кезде ``OSError: [Errno 98]`` немесе ``OSError: [WinError 10013]`` көресіз. :ref:`address-already-in-use` онымен қалай күресуге болады екенін қараңыз.

:doc:`database` - ге өтіңіз.
