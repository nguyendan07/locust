###
API
###


User class
============

.. autoclass:: locust.User
    :members: wait_time, tasks, weight, fixed_count, abstract, on_start, on_stop, wait, context, environment

HttpUser class
================

.. autoclass:: locust.HttpUser
    :members: wait_time, tasks, client, abstract

FastHttpUser class
==================

.. autoclass:: locust.contrib.fasthttp.FastHttpUser
    :members: wait_time, tasks, client, abstract, rest
    :noindex:

TaskSet class
=============

.. autoclass:: locust.TaskSet
    :members: user, parent, wait_time, client, tasks, interrupt, schedule_task, on_start, on_stop, wait

task decorator
==============

.. autofunction:: locust.task

tag decorator
==============

.. autofunction:: locust.tag

SequentialTaskSet class
=======================

.. autoclass:: locust.SequentialTaskSet
    :members: user, parent, wait_time, client, tasks, interrupt, schedule_task, on_start, on_stop


.. _wait_time_functions:

Built in wait_time functions
============================

.. automodule:: locust.wait_time
    :members: between, constant, constant_pacing, constant_throughput

HttpSession class
=================

.. autoclass:: locust.clients.HttpSession
    :members: __init__, request, get, post, delete, put, head, options, patch

Response class
==============

Lớp này thực sự nằm trong thư viện `requests <https://requests.readthedocs.io/>`_,
vì đó là thứ Locust sử dụng để thực hiện các yêu cầu HTTP, nhưng nó được bao gồm trong tài liệu API
để locust vì nó quan trọng khi viết các bài kiểm tra tải locust. Bạn cũng có thể xem
lớp :py:class:`Response <requests.Response>` tại `requests documentation <https://requests.readthedocs.io/>`_.

.. autoclass:: requests.Response
    :inherited-members:
    :noindex:

ResponseContextManager class
============================

.. autoclass:: locust.clients.ResponseContextManager
    :members: success, failure


.. _exceptions:

Exceptions
==========

.. autoexception:: locust.exception.InterruptTaskSet


.. autoexception:: locust.exception.RescheduleTask


.. autoexception:: locust.exception.RescheduleTaskImmediately


Environment class
=================
.. autoclass:: locust.env.Environment
    :members:


.. _events:

Event hooks
===========

Locust cung cấp các hooks sự kiện mà có thể được sử dụng để mở rộng Locust theo nhiều cách.

Các hook sự kiện sau đây có sẵn dưới :py:attr:`Environment.events <locust.env.Environment.events>`,
và cũng có một tham chiếu đến các sự kiện này dưới ``locust.events`` mà có thể được sử dụng ở mức độ module
của các tập lệnh locust (vì một Environment instance chưa được tạo khi tập lệnh locust được nhập).

.. autoclass:: locust.event.Events
    :members:


.. note::

    Nó được khuyến khích rằng bạn thêm một wildcard keyword argument trong các trình nghe sự kiện của bạn
    để ngăn mã của bạn bị hỏng nếu các đối số mới được thêm trong một phiên bản tương lai.

EventHook class
---------------

The event hooks are instances of the **locust.events.EventHook** class:

.. autoclass:: locust.event.EventHook
    :members:

Runner classes
=====================

.. autoclass:: locust.runners.Runner
    :members: start, stop, quit, user_count

.. autoclass:: locust.runners.LocalRunner

.. autoclass:: locust.runners.MasterRunner
    :members: register_message, send_message

.. autoclass:: locust.runners.WorkerRunner
    :members: register_message, send_message, client_id, worker_index

Web UI class
============

.. autoclass:: locust.web.WebUI
    :members:

Other
=====

.. autoclass:: locust.shape.LoadTestShape
    :members:

.. autoclass:: locust.stats.RequestStats
    :members: get

.. autoclass:: locust.stats.StatsEntry

.. autofunction:: locust.debug.run_single_user
