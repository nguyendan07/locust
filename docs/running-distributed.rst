.. _running-distributed:

===========================
Tải trọng phân tán (Distributed load generation)
===========================

Một quy trình chạy Locust có thể mô phỏng một lưu lượng khá cao. Đối với một kế hoạch kiểm thử đơn giản và dữ liệu gói nhỏ, nó có thể tạo hơn một nghìn yêu cầu mỗi giây, có thể hơn mười nghìn nếu bạn sử dụng :ref:`FastHttpUser <increase-performance>`.

Nhưng nếu kế hoạch kiểm thử của bạn phức tạp hoặc bạn muốn chạy thêm nhiều tải, bạn cần mở rộng ra nhiều quy trình, có thể là nhiều máy. May mắn thay, Locust hỗ trợ chạy phân tán ngay từ đầu.

Để làm điều này, bạn bắt đầu một phiên bản của Locust với cờ ``--master`` và một hoặc nhiều sử dụng cờ ``--worker``. Phiên bản master chạy giao diện web của Locust, và thông báo cho các phiên bản worker khi nào để tạo/đóng người dùng. Các phiên bản worker chạy người dùng của bạn và gửi thống kê trở lại master. Phiên bản master không chạy bất kỳ người dùng nào.

Để đơn giản hóa việc khởi động, bạn có thể sử dụng cờ ``--processes``. Nó sẽ khởi chạy một quy trình master và số lượng quy trình worker được chỉ định. Nó cũng có thể được sử dụng kết hợp với ``--worker``, sau đó nó chỉ sẽ khởi chạy worker. Tính năng này phụ thuộc vào `fork() <https://linux.die.net/man/3/fork>`_ nên nó không hoạt động trên Windows.

.. note::
    Bởi vì Python không thể tận dụng đầy đủ hơn một lõi cho mỗi quy trình (xem `GIL <https://realpython.com/python-gil/>`_), bạn cần chạy một phiên bản worker cho mỗi lõi bộ xử lý để truy cập vào tất cả năng lực tính toán của bạn.

.. note::
    Gần như không giới hạn số lượng người dùng bạn có thể chạy trên mỗi worker. Locust/gevent có thể chạy hàng nghìn hoặc thậm chí hàng chục nghìn người dùng trên mỗi quy trình một cách tốt, miễn là tổng số lượng yêu cầu của họ (RPS) không quá cao.

    Nếu Locust *gần như* cạn kiệt tài nguyên CPU, nó sẽ ghi lại một cảnh báo. Nếu không có cảnh báo nhưng bạn vẫn không thể tạo ra tải trọng dự kiến, thì vấn đề phải là :ref:`increaserr`.

Máy đơn (Single machine)
==============

Rất đơn giản để khởi chạy một quy trình master và 4 quy trình worker::

    locust --processes 4

Bạn thậm chí có thể tự động phát hiện số lõi logic trong máy của bạn và khởi chạy một worker cho mỗi lõi::
    
        locust --processes -1
    
Nhiều máy (Multiple machines)
=================

Bắt đầu locust ở chế độ master trên một máy::

    locust -f my_locustfile.py --master

Và sau đó trên mỗi máy worker:

.. code-block:: bash

    locust -f - --worker --master-host <your master> --processes 4

.. note::
    Đối số ``-f -`` cho biết Locust lấy locustfile từ master thay vì từ hệ thống tệp cục bộ của nó. Tính năng này được giới thiệu trong Locust 2.23.0.

Nhiều máy, sử dụng locust-swarm (Multiple machines, using locust-swarm)
=====================================

Khi bạn thay đổi locustfile, bạn cần khởi động lại tất cả các quy trình Locust. `locust-swarm <https://github.com/SvenskaSpel/locust-swarm>`_ tự động hóa điều này cho bạn. Nó cũng giải quyết vấn đề truy cập qua tường lửa/mạng từ worker đến master bằng cách sử dụng các đường hầm SSH (điều này thường là một vấn đề nếu master đang chạy trên máy làm việc của bạn và worker đang chạy trong một trung tâm dữ liệu nào đó).

.. code-block:: bash

    pip install locust-swarm

    swarm -f my_locustfile.py --loadgen-list worker-server1,worker-server2 <any other regular locust parameters>

Xem `locust-swarm <https://github.com/SvenskaSpel/locust-swarm>`_ để biết thêm chi tiết.

Tùy chọn cho tải trọng phân tán (Options for distributed load generation)
=======================================

``--master-host <hostname or ip>``
----------------------------------

Tùy chọn được sử dụng tùy chọn với ``--worker`` để đặt tên máy chủ của master (mặc định là localhost)

``--master-port <port number>``
-------------------------------

Tùy chọn được sử dụng tùy chọn với ``--worker`` để đặt số cổng của master (mặc định là 5557).

``--master-bind-host <ip>``
---------------------------

Tùy chọn được sử dụng tùy chọn với ``--master``. Xác định giao diện mạng mà master sẽ kết nối. Mặc định là * (tất cả các giao diện có sẵn).

``--master-bind-port <port number>``
------------------------------------

Tùy chọn được sử dụng tùy chọn với ``--master``. Xác định cổng mạng mà master sẽ lắng nghe. Mặc định là 5557.

``--expect-workers <number of workers>``
----------------------------------------

Sử dụng khi bắt đầu master với ``--headless``. Master sẽ chờ cho đến khi X worker kết nối trước khi bắt đầu kiểm thử.

Giao tiếp giữa các nút (Communicating across nodes)
=============================================

Khi chạy Locust ở chế độ phân tán, bạn có thể muốn giao tiếp giữa các nút master và worker để phối hợp kiểm thử. Điều này có thể dễ dàng thực hiện với các thông điệp tùy chỉnh sử dụng các kết nối thông điệp được tích hợp:

.. code-block:: python

    from locust import events
    from locust.runners import MasterRunner, WorkerRunner

    # Được kích hoạt khi worker nhận được tin nhắn có kiểu (message of type) 'test_users'
    def setup_test_users(environment, msg, **kwargs):
        for user in msg.data:
            print(f"User {user['name']} received")
        environment.runner.send_message('acknowledge_users', f"Thanks for the {len(msg.data)} users!")

    # Được kích hoạt khi master nhận được tin nhắn có kiểu (message of type) 'acknowledge_users'
    def on_acknowledge(msg, **kwargs):
        print(msg.data)

    @events.init.add_listener
    def on_locust_init(environment, **_kwargs):
        if not isinstance(environment.runner, MasterRunner):
            environment.runner.register_message('test_users', setup_test_users)
        if not isinstance(environment.runner, WorkerRunner):
            environment.runner.register_message('acknowledge_users', on_acknowledge)

    @events.test_start.add_listener
    def on_test_start(environment, **_kwargs):
        if not isinstance(environment.runner, WorkerRunner):
            users = [
                {"name": "User1"},
                {"name": "User2"},
                {"name": "User3"},
            ]
            environment.runner.send_message('test_users', users)  

Lưu ý rằng khi chạy cục bộ (tức là không phân tán), chức năng này sẽ được bảo lưu;
các tin nhắn sẽ đơn giản được xử lý bởi runner gửi chúng.

.. note::
    Sử dụng các tùy chọn mặc định khi đăng ký một trình xử lý tin nhắn sẽ chạy hàm nghe trong một cách **chặn**, dẫn đến việc trì hoãn tin nhắn heartbeat và các tin nhắn khác trong thời gian thực thi.
    Nếu bạn nghĩ rằng trình xử lý tin nhắn của mình sẽ cần chạy trong hơn một giây thì bạn có thể đăng ký nó
    như **concurrent**. Locust sẽ khiến nó chạy trong greenlet riêng của nó. Lưu ý rằng các greenlet này sẽ không bao giờ được join():ed.

    .. code-block::

        environment.runner.register_message('test_users', setup_test_users, concurrent=True)

Để biết thêm chi tiết, xem `ví dụ hoàn chỉnh <https://github.com/locustio/locust/tree/master/examples/custom_messages.py>`_.


Chạy phân tán với Docker (Running distributed with Docker)
=============================================

Xem :ref:`running-in-docker`


Chạy Locust phân tán mà không có giao diện web (Running Locust distributed without the web UI)
=============================================

Xem :ref:`running-distributed-without-web-ui`


Tăng hiệu suất của Locust (Increase Locust's performance)
=============================

Nếu bạn đang lên kế hoạch chạy các bài kiểm tra tải lớn, bạn có thể quan tâm đến việc sử dụng máy khách HTTP thay thế được giao với Locust. Bạn có thể đọc thêm về nó ở đây: :ref:`increase-performance`.
