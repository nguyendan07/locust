.. _testing-other-systems:

===============================
Kiểm tra các hệ thống/giao thức khác (Testing other systems/protocols)
===============================

Locust chỉ đi kèm với hỗ trợ tích hợp cho HTTP/HTTPS nhưng nó có thể được mở rộng để kiểm tra hầu hết các hệ thống. Thông thường điều này được thực hiện bằng cách bọc thư viện giao thức và kích hoạt một sự kiện :py:attr:`request <locust.event.Events.request>` sau mỗi cuộc gọi đã hoàn thành, để cho Locust biết điều gì đã xảy ra.

.. note::

    Quan trọng là thư viện giao thức bạn sử dụng có thể được `monkey-patched <https://www.gevent.org/intro.html#monkey-patching>`_ bởi gevent. 
    
    Hầu hết các thư viện là Python thuần (sử dụng mô-đun Python ``socket`` hoặc một hàm thư viện chuẩn khác như ``subprocess``) nên sẽ hoạt động tốt ngay - nhưng nếu chúng thực hiện các cuộc gọi I/O từ mã đã biên dịch C, gevent sẽ không thể patch nó. Điều này sẽ chặn toàn bộ quá trình Locust/Python (thực tế giới hạn bạn chỉ có thể chạy một Người dùng cho mỗi quá trình worker).

    Một số thư viện C cho phép các phương pháp khác để giải quyết vấn đề. Ví dụ, nếu bạn muốn sử dụng psycopg2 để kiểm tra hiệu suất PostgreSQL, bạn có thể sử dụng `psycogreen <https://github.com/psycopg/psycogreen/>`_. Nếu bạn sẵn lòng làm việc, bạn có thể tự patch một thư viện, nhưng đó là ngoài phạm vi của tài liệu này.

XML-RPC
=======

Giả sử chúng ta có một máy chủ XML-RPC mà chúng ta muốn kiểm tra tải.

.. literalinclude:: ../examples/custom_xmlrpc_client/server.py

Chúng ta có thể xây dựng một khách hàng XML-RPC chung, bằng cách bọc :py:class:`xmlrpc.client.ServerProxy`.

.. literalinclude:: ../examples/custom_xmlrpc_client/xmlrpc_locustfile.py

gRPC
====

Giả sử chúng ta có một máy chủ `gRPC <https://github.com/grpc/grpc>`_ mà chúng ta muốn kiểm tra tải:

.. literalinclude:: ../examples/grpc/hello_server.py

Lớp cơ sở GrpcUser chung gửi sự kiện đến Locust bằng một `interceptor <https://pypi.org/project/grpc-interceptor/>`_:

.. literalinclude:: ../examples/grpc/grpc_user.py

Và một tệp locust sử dụng ở trên sẽ trông như thế này:

.. literalinclude:: ../examples/grpc/locustfile.py

.. _testing-request-sdks:

Các thư viện/SDK dựa trên requests
=============================

Nếu bạn muốn sử dụng một thư viện sử dụng đối tượng `requests.Session <https://requests.readthedocs.io/en/latest/api/#requests.Session>`_ dưới lớp, bạn có thể bỏ qua tất cả sự phức tạp ở trên.

Một số thư viện cho phép bạn truyền một Session một cách rõ ràng, như ví dụ là khách hàng SOAP được cung cấp bởi `Zeep <https://docs.python-zeep.org/en/master/transport.html#tls-verification>`_. Trong trường hợp đó, chỉ cần truyền cho nó :py:attr:`client <locust.HttpUser.client>` của ``HttpUser`` của bạn, và bất kỳ yêu cầu nào được thực hiện bằng thư viện sẽ được ghi nhật ký trong Locust.

Ngay cả khi thư viện của bạn không tiết lộ điều đó trong giao diện của nó, bạn có thể làm cho nó hoạt động bằng cách ghi đè một số Session được sử dụng bên trong. Dưới đây là một ví dụ về cách thực hiện điều đó cho khách hàng `Archivist <https://github.com/jitsuin-inc/archivist-python>`_.

.. literalinclude:: ../examples/sdk_session_patching/session_patch_locustfile.py

REST
====

Xem :ref:`FastHttpUser <rest>`

Ví dụ khác
==============

Xem `locust-plugins <https://github.com/SvenskaSpel/locust-plugins#users>`_ nó có người dùng cho WebSocket/SocketIO, Kafka, Selenium/WebDriver, Playwright và nhiều hơn nữa.
