.. currentmodule:: flask

Блог жүргізу схемасы
==============

Блог схемасын жасау үшін аутентификация схемасын жазу кезінде білген әдістерді қолданасыз. Блогта барлық жазбалар тізімделуі керек, авторизацияланған пайдаланушыларға жазбалар жасауға және жазба авторына оны өңдеуге немесе жоюға рұқсат етілуі керек.

Әрбір көрініс іске асырылған кезде әзірлеу серверінің жұмысын қолдаңыз. Өзгерістерді сақтаған кезде, браузердегі URL мекенжайына өтіп, оларды тексеріп көріңіз.

Схемалар
-------------

Схеманы анықтаңыз және оны қолданбалар зауытына тіркеңіз.

.. code-block:: python
    :caption: ``flaskr/blog.py``

    from flask import (
        Blueprint, flash, g, redirect, render_template, request, url_for
    )
    from werkzeug.exceptions import abort

    from flaskr.auth import login_required
    from flaskr.db import get_db

    bp = Blueprint('blog', __name__)

Өндірушіден сызбаны импорттаңыз және тіркеңіз :meth:`app.register_blueprint()<Flask.register_blueprint>`. Қолданбаны қайтармас бұрын зауыттық мүмкіндіктің соңына жаңа кодты қойыңыз.

.. code-block:: python
    :caption: ``flaskr/__init__.py``

    def create_app():
        app = ...
        # existing code omitted

        from . import blog
        app.register_blueprint(blog.bp)
        app.add_url_rule('/', endpoint='index')

        return app


Аутентификация схемасынан айырмашылығы, блог схемасында ``url_prefix``жоқ. Осылайша, ``index`` көрінісі  ``/`` ішінде болады, ``/create`` көрінісі  ``/create`` және т.б. Блог, Flask-тің негізгі функциясы,сондықтан блог индексі негізгі индекс болады.

бірдей URL мекенжайын жасау ``/``кез келген жағдайда. Басқа қолданбада сіз ``url_prefix`` блог схемасын тағайындап, ``hello`` көрінісіне ұқсас қосымшалар фабрикасындағы ``index``жеке көрінісін анықтай аласыз. Содан кейін ``index`` және ``blog.index`` соңғы нүктелері мен URL мекенжайлары әр түрлі болар еді.

Индекс
-----

Индексте барлық жазбалар көрсетіледі, алдымен ең бірінші. ``JOIN``нәтижесінде ``user`` кестесінен автор туралы ақпарат алу үшін қолданылады.

.. code-block:: python
    :caption: ``flaskr/blog.py``

    @bp.route('/')
    def index():
        db = get_db()
        posts = db.execute(
            'SELECT p.id, title, body, created, author_id, username'
            ' FROM post p JOIN user u ON p.author_id = u.id'
            ' ORDER BY created DESC'
        ).fetchall()
        return render_template('blog/index.html', posts=posts)

.. code-block:: html+jinja
    :caption: ``flaskr/templates/blog/index.html``

    {% extends 'base.html' %}

    {% block header %}
      <h1>{% block title %}Posts{% endblock %}</h1>
      {% if g.user %}
        <a class="action" href="{{ url_for('blog.create') }}">New</a>
      {% endif %}
    {% endblock %}

    {% block content %}
      {% for post in posts %}
        <article class="post">
          <header>
            <div>
              <h1>{{ post['title'] }}</h1>
              <div class="about">by {{ post['username'] }} on {{ post['created'].strftime('%Y-%m-%d') }}</div>
            </div>
            {% if g.user['id'] == post['author_id'] %}
              <a class="action" href="{{ url_for('blog.update', id=post['id']) }}">Edit</a>
            {% endif %}
          </header>
          <p class="body">{{ post['body'] }}</p>
        </article>
        {% if not loop.last %}
          <hr>
        {% endif %}
      {% endfor %}
    {% endblock %}

Пайдаланушы кірген кезде ``header`` блогы ``create``көрінісіне сілтеме қосады. Пайдаланушы жарияланымның авторы болған кезде, ол "Edit" сілтемесін ``update``  сол жарияланым үшін. ``loop.last`` - бұл `Jinja for loops`_ішінде қол жетімді арнайы айнымалы. Ол әр хабарламадан кейін жолды көрсету үшін қолданылады, соңғысынан басқа, оларды көзбен бөлу үшін.

.. _Jinja for loops: https://jinja.palletsprojects.com/templates/#for


Жасау
------

``create`` көрінісі авторизация үшін тіркеу ``register`` сияқты жұмыс істейді. Пішін көрсетіледі немесе жарияланған деректер тексеріледі және жазба дерекқорға қосылады немесе қате туралы хабар көрсетіледі.

Сіз бұрын жазған ``login_required`` декоры блог көріністерінде қолданылады. Пайдаланушы осы көріністерге кіру үшін жүйеге кіруі керек, әйтпесе ол кіру бетіне қайта бағытталады.

.. code-block:: python
    :caption: ``flaskr/blog.py``

    @bp.route('/create', methods=('GET', 'POST'))
    @login_required
    def create():
        if request.method == 'POST':
            title = request.form['title']
            body = request.form['body']
            error = None

            if not title:
                error = 'Title is required.'

            if error is not None:
                flash(error)
            else:
                db = get_db()
                db.execute(
                    'INSERT INTO post (title, body, author_id)'
                    ' VALUES (?, ?, ?)',
                    (title, body, g.user['id'])
                )
                db.commit()
                return redirect(url_for('blog.index'))

        return render_template('blog/create.html')

.. code-block:: html+jinja
    :caption: ``flaskr/templates/blog/create.html``

    {% extends 'base.html' %}

    {% block header %}
      <h1>{% block title %}New Post{% endblock %}</h1>
    {% endblock %}

    {% block content %}
      <form method="post">
        <label for="title">Title</label>
        <input name="title" id="title" value="{{ request.form['title'] }}" required>
        <label for="body">Body</label>
        <textarea name="body" id="body">{{ request.form['body'] }}</textarea>
        <input type="submit" value="Save">
      </form>
    {% endblock %}


Жаңарту
------

 ``update``көріністерінде де, ``delete``көріністерінде ``id``  идентификатормен ``post`` алу және автордың тіркелген пайдаланушымен сәйкес келетіндігін тексеру қажет болады. . Кодтың қайталануын болдырмау үшін Сіз ``post``алу үшін функция жазып, оны әр көріністен шақыра аласыз.

.. code-block:: python
    :caption: ``flaskr/blog.py``

    def get_post(id, check_author=True):
        post = get_db().execute(
            'SELECT p.id, title, body, created, author_id, username'
            ' FROM post p JOIN user u ON p.author_id = u.id'
            ' WHERE p.id = ?',
            (id,)
        ).fetchone()

        if post is None:
            abort(404, f"Post id {id} doesn't exist.")

        if check_author and post['author_id'] != g.user['id']:
            abort(403)

        return post

:func:`abort` - HTTP күй кодын қайтаратын арнайы ерекшелікті шақырады. Қате туралы хабарды көрсету үшін қосымша хабарлама қажет, әйтпесе әдепкі хабарлама қолданылады.``404``"табылмады" дегенді білдіреді, ал``403``тыйым салынған"дегенді білдіреді. (``401`` "рұқсат етілмеген" дегенді білдіреді, бірақ сіз бұл күйді қайтарудың орнына кіру бетіне бағыттайсыз.)

``Check_author`` аргументі автордың тексеруінсіз ``post`` алу үшін функцияны қолдануға болатындай етіп анықталған. Егер сіз жеке жазбаны пайдаланушы маңызды емес бетте көрсету үшін көрініс жазсаңыз, бұл пайдалы болар еді, себебі ол жазбаны өзгертпейді.

.. code-block:: python
    :caption: ``flaskr/blog.py``

    @bp.route('/<int:id>/update', methods=('GET', 'POST'))
    @login_required
    def update(id):
        post = get_post(id)

        if request.method == 'POST':
            title = request.form['title']
            body = request.form['body']
            error = None

            if not title:
                error = 'Title is required.'

            if error is not None:
                flash(error)
            else:
                db = get_db()
                db.execute(
                    'UPDATE post SET title = ?, body = ?'
                    ' WHERE id = ?',
                    (title, body, id)
                )
                db.commit()
                return redirect(url_for('blog.index'))

        return render_template('blog/update.html', post=post)

Осы уақытқа дейін жазған көріністерден айырмашылығы, ``update`` функциясы ``id``аргументін қабылдайды. Бұл маршрутта ``<int:id>`` сәйкес келеді. Нақты URL мекенжайы ``/1/update`` сияқты болады. Flask  ``1``түзетеді, оның :class:`int` екеніне көз жеткізіңіз және оны ``id``аргументі ретінде жіберіңіз. Егер сіз  ``int:``көрсетпесеңіз және оның орнына ``<id>`` жасасаңыз, бұл жол болады.
Жаңарту бетінің URL мекенжайын жасау үшін :func:`url_for`не толтыру керектігін білу үшін ``id`` жіберілуі керек: ``url_for('blog.update', id=post['id'])``. Бұл ``index.html`` де бар жоғарыдағы файлы.

``create``және ``update`` көріністері өте ұқсас. Негізгі айырмашылық - '``update`` көрінісі ``INSERT`` орнына ``post`` нысанын және ``UPDATE`` сұрауын қолданады. Кейбір ақылды рефакторинг арқылы сіз екі әрекет үшін де бір көрініс пен үлгіні пайдалана аласыз, бірақ Нұсқаулық үшін оларды бөлек ұстау түсінікті.

.. code-block:: html+jinja
    :caption: ``flaskr/templates/blog/update.html``

    {% extends 'base.html' %}

    {% block header %}
      <h1>{% block title %}Edit "{{ post['title'] }}"{% endblock %}</h1>
    {% endblock %}

    {% block content %}
      <form method="post">
        <label for="title">Title</label>
        <input name="title" id="title"
          value="{{ request.form['title'] or post['title'] }}" required>
        <label for="body">Body</label>
        <textarea name="body" id="body">{{ request.form['body'] or post['body'] }}</textarea>
        <input type="submit" value="Save">
      </form>
      <hr>
      <form action="{{ url_for('blog.delete', id=post['id']) }}" method="post">
        <input class="danger" type="submit" value="Delete" onclick="return confirm('Are you sure?');">
      </form>
    {% endblock %}

Бұл үлгінің екі формасы бар. Біріншісі өңделген деректерді ағымдағы бетте жариялайды (``/<id>/update``). Басқа формада тек батырма бар және оның орнына жою көрінісіне жіберілетін ``action`` атрибутын көрсетеді. Түйме жібермес бұрын растау тілқатысу терезесін көрсету үшін кейбір JavaScript пайдаланады.

Үлгі ``{{ request.form['title'] or post['title'] }}``  пішінде қандай деректер көрсетілетінін таңдау үшін қолданылады. Пішін жіберілмеген кезде``post`` бастапқы деректері көрсетіледі, бірақ егер дұрыс емес пішін деректері жарияланған болса, пайдаланушы қатені түзете алатындай етіп оны көрсеткіңіз келеді, сондықтан оның орнына ``request.form``қолданылады. :data:`request`  - бұл шаблондарда автоматты түрде қол жетімді тағы бір айнымалы.


Жою
------

Жою көрінісінің жеке үлгісі жоқ, жою  түймесі ``update.html``бөлігі болып табылады және URL мекенжайына хабарламалар жібереді ``/<id>/delete``. Үлгі болмағандықтан, ол тек ``POST``әдісін өңдейді, содан кейін `index`көрінісіне бағыттайды.

.. code-block:: python
    :caption: ``flaskr/blog.py``

    @bp.route('/<int:id>/delete', methods=('POST',))
    @login_required
    def delete(id):
        get_post(id)
        db = get_db()
        db.execute('DELETE FROM post WHERE id = ?', (id,))
        db.commit()
        return redirect(url_for('blog.index'))

Құттықтаймыз, сіз өз өтінішіңізді жазуды аяқтадыңыз! Барлығын шолғышта сынап көруге біраз уақыт бөліңіз. Дегенмен, жоба аяқталғанға дейін әлі көп нәрсе істеу керек.

Келесіге өтіңіз :doc:`install`.
