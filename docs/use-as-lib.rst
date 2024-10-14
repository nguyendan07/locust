.. _use-as-lib:

==========================
Sử dụng Locust như một thư viện
==========================

Locust có thể được sử dụng như một thư viện Python bình thường, thay vì chạy từ command line. Điều này cho phép bạn tạo ra các test case của mình, và chạy chúng từ một script Python.

Bắt đầu bằng cách tạo một instance :py:class:`Environment <locust.env.Environment>`:

.. code-block:: python

    from locust.env import Environment
    
    env = Environment(user_classes=[MyTestUser])

Instance :py:class:`Environment <locust.env.Environment>` có thể sử dụng
:py:meth:`create_local_runner <locust.env.Environment.create_local_runner>`,
:py:meth:`create_master_runner <locust.env.Environment.create_master_runner>` để bắt đầu một instance
:py:class:`Runner <locust.runners.Runner>`, mà có thể được sử dụng để bắt đầu một load test:

.. code-block:: python

    env.create_local_runner()
    env.runner.start(5000, spawn_rate=20)
    env.runner.greenlet.join()

Cũng có thể bỏ qua logic dispatch và distribution, và điều khiển thủ công các user được spawn:

.. code-block:: python

    new_users = env.runner.spawn_users({MyUserClass.__name__: 2})
    new_users[1].my_custom_token = "custom-token-2"
    new_users[0].my_custom_token = "custom-token-1"

Ví dụ trên chỉ hoạt động trong standalone/local runner mode và là một tính năng thử nghiệm. Một cách tiếp cận phổ biến/hợp lý hơn sẽ là sử dụng sự kiện ``init`` hoặc ``test_start`` để lấy/tạo một danh sách các token và sử dụng :ref:`on-start-on-stop` để đọc từ danh sách đó và thiết lập chúng trên các User instances của bạn.

.. note::

    Mặc dù có thể tạo ra các locust workers theo cách này (sử dụng :py:meth:`create_worker_runner <locust.env.Environment.create_worker_runner>`), điều đó hầu như không bao giờ hợp lý. Mỗi worker cần phải ở trong một Python process riêng và tương tác trực tiếp với worker runner có thể làm hỏng mọi thứ. Chỉ cần khởi chạy workers bằng cách sử dụng lệnh ``locust --worker ...`` thông thường.

Cũng có thể sử dụng :py:class:`Environment <locust.env.Environment>` instance's
:py:meth:`create_web_ui <locust.env.Environment.create_web_ui>` để bắt đầu một Web UI mà có thể được sử dụng
để xem các stats, và điều khiển runner (ví dụ: bắt đầu và dừng load test):

.. code-block:: python

    env.create_local_runner()
    env.create_web_ui()
    env.web_ui.greenlet.join()

Bỏ qua monkey patching (Skipping monkey patching)
========================

Một số gói như boto3 có thể không tương thích khi sử dụng Locust như một thư viện, nơi monkey patching đã được áp dụng. Trong trường hợp này monkey patching có thể được vô hiệu hóa bằng cách thiết lập ``LOCUST_SKIP_MONKEY_PATCH=1`` như biến môi trường.

Ví dụ đầy đủ
============

.. literalinclude:: ../examples/use_as_lib.py
    :language: python
