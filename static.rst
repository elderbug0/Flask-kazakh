Cтатикалық файлдар
============

Түпнұсқалық растама көріністері мен үлгілері жұмыс істейді, бірақ қазір олар өте қарапайым көрінеді. HTML орналасуына стиль қосу үшін кейбір `CSS`_ қосылуы мүмкін. Стиль өзгермейді, сондықтан ол *статикалық* файл емес
шаблон болады.

Flask, ``flask/static`` каталогына қатысты жолды алатын және оған қызмет көрсететін ``static`` көрінісін автоматты түрде қосады. ``base.html`` үлгісінде ``style.css`` файлына сілтеме әлдеқашан бар:

.. code-block:: html+jinja

    {{ url_for('static', filename='style.css') }}

CSS-тен басқа, статикалық файлдардың басқа түрлері JavaScript-ті функциялар немесе логотип суреттері болуы мүмкін. Олардың барлығы 
``flaskr/static`` астына орналастырылған және сілтемесі 
``url_for('static', filename='...')``. болады

Бұл оқулық CSS жазу жолына арналмаған, сондықтан сіз жай ғана көшіре аласыз.
Келесіні ``flaskr/static/style.css`` файлына енгізіңіз:

.. code-block:: css
    :caption: ``flaskr/static/style.css``

    html { font-family: sans-serif; background: #eee; padding: 1rem; }
    body { max-width: 960px; margin: 0 auto; background: white; }
    h1 { font-family: serif; color: #377ba8; margin: 1rem 0; }
    a { color: #377ba8; }
    hr { border: none; border-top: 1px solid lightgray; }
    nav { background: lightgray; display: flex; align-items: center; padding: 0 0.5rem; }
    nav h1 { flex: auto; margin: 0; }
    nav h1 a { text-decoration: none; padding: 0.25rem 0.5rem; }
    nav ul  { display: flex; list-style: none; margin: 0; padding: 0; }
    nav ul li a, nav ul li span, header .action { display: block; padding: 0.5rem; }
    .content { padding: 0 1rem 1rem; }
    .content > header { border-bottom: 1px solid lightgray; display: flex; align-items: flex-end; }
    .content > header h1 { flex: auto; margin: 1rem 0 0.25rem 0; }
    .flash { margin: 1em 0; padding: 1em; background: #cae6f6; border: 1px solid #377ba8; }
    .post > header { display: flex; align-items: flex-end; font-size: 0.85em; }
    .post > header > div:first-of-type { flex: auto; }
    .post > header h1 { font-size: 1.5em; margin-bottom: 0; }
    .post .about { color: slategray; font-style: italic; }
    .post .body { white-space: pre-line; }
    .content:last-child { margin-bottom: 0; }
    .content form { margin: 1em 0; display: flex; flex-direction: column; }
    .content label { font-weight: bold; margin-bottom: 0.5em; }
    .content input, .content textarea { margin-bottom: 1em; }
    .content textarea { min-height: 12em; resize: vertical; }
    input.danger { color: #cc2f2e; }
    input[type=submit] { align-self: start; min-width: 10em; }

``style.css``-тын ықшам нұсқасын мына жерден таба аласыз
:gh:`example code <examples/tutorial/flaskr/static/style.css>`.


http://127.0.0.1:5000/auth/login өтіңіз және төмендегі скриншот келесідей болуы керек.


.. image:: flaskr_login.png
    :align: center
    :class: screenshot
    :alt: screenshot of login page

CSS туралы қосымша ақпарат - `Mozilla's documentation <CSS_>`_. Егер
статикалық файлды өзгертсеңіз, браузер бетін жаңартыңыз. Өзгеріс болмаса, браузердің кэшін тазалап көріңіз.

.. _CSS: https://developer.mozilla.org/docs/Web/CSS

:doc:`blog`. - на өтіңіз.


