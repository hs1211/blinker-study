=============
Blinker Usage
=============

Blinker는 오브젝트 간 혹은 오브젝트에서의 브로드 캐스트하는 시그널을 보내기 위하여 사용한다.


Feature
-------

- global registry of named signals
- Anonymous signal
- custom name registries
- permanently or temporarily connected receivers
- auto disconnect via weak referencing
- sending arbitrary data payloads
- thread safe


Usage
------

간단한 예제를 먼저 살펴보자.

1. 시그널 생성하기

.. code-block:: python

    from blinker import signal
    init = signal('initialized')


2.  시그널 구독하기

    아래처럼 시그널을 구독하기 위해서는 콜백함수를 등록해야 한다
    이때 콜백함수의 파라미터는 sender의 object가 된다

.. code-block:: python

    from blinker import signal

    def subscriber(sender):
        print("Got a signal sent by %r" % sender)

    ready = signal('ready')
    ready.connect(subscriber)

3.  시그널 보내기

    아래처럼 시그널을 독하기 위해서는 콜백함수를 등록해야 한다
    이때 콜백함수의 파라미터는 sender의 object가 된다

.. code-block::

    from blinker import signal
    ready = signal('ready')
    ready.send(self)

    > Got a signal sent by <Processor a>


시그널을 통한 송수신 예제
-------------------

.. code-block:: shell

    >>> send_data = signal('send-data')
    >>> @send_data.connect
    ... def receive_data(sender, **kw):
    ...     print("Caught signal from %r, data %r" % (sender, kw))
    ...     return 'received!'
    ...
    >>> result = send_data.send('anonymous', abc=123)
    Caught signal from 'anonymous', data {'abc': 123}


Flask APP
---------

- app/__init__.py

.. code-block:: python

    # generate signal
    from blinker import Namespace

    socializer_signals = Namespace()
    user_followed = socializer_signals.signal('user-followed')

    # register handler
    from signal_handlers import connect_handlers
    connect_handlers()


- app/signal_handlers.py:

.. code-block:: python

    # handler
    def user_followed_email(user, **kwargs):
        logger.debug("Send an email to {user}".format(user=user.username))

    # subscribing handler
    from application import user_followed
    def connect_handlers():
        user_followed.connect(user_followed_email)


- app/user/user.py

.. code-block:: python

    def follow(self, user):
        """Follow the given user.
        Return `False` if the user was already following the user.
        """
        if self.is_following(user):
            return False
        self.followed.append(user)
        # Publish the signal event using the current model (self) as sender.
        user_followed.send(self)
        return self


LICENSE
-------

MIT License.