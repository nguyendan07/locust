.. _running-in-debugger:

===========================
Chạy thử nghiệm trong trình gỡ lỗi (Running tests in a debugger)
===========================

Chạy Locust trong trình gỡ lỗi rất hữu ích khi phát triển các bài kiểm tra của bạn. Trong số những điều khác, bạn có thể kiểm tra một phản hồi cụ thể hoặc kiểm tra một biến thể của User.

Nhưng trình gỡ lỗi đôi khi gặp vấn đề với các ứng dụng gevent phức tạp như Locust, và có rất nhiều điều đang diễn ra trong framework mà bạn có lẽ không quan tâm. Để đơn giản hóa điều này, Locust cung cấp một phương thức được gọi là :py:func:`run_single_user <locust.debug.run_single_user>`:

.. literalinclude:: ../examples/debugging.py
    :language: python

Nó ngầm định đăng ký một trình xử lý sự kiện cho sự kiện :ref:`request <extending_locust>` để in một số thống kê về mỗi yêu cầu được thực hiện:

.. code-block:: console

    type    name                                           resp_ms exception
    GET     /hello                                         38      ConnectionRefusedError(61, 'Connection refused')
    GET     /hello                                         4       ConnectionRefusedError(61, 'Connection refused')

Bạn có thể cấu hình chính xác những gì được in bằng cách chỉ định các tham số cho :py:func:`run_single_user <locust.debug.run_single_user>`.

Hãy chắc chắn rằng bạn đã kích hoạt gevent trong cài đặt trình gỡ lỗi của bạn. Trong ``launch.json`` của VS Code, nó trông như thế này:

.. literalinclude:: ../.vscode/launch.json
    :language: json

Nếu bạn muốn chạy toàn bộ runtime Locust (với ramp up, phân tích dòng lệnh vv), bạn cũng có thể làm điều đó:

.. literalinclude:: ../.vscode/launch_locust.json
    :language: json

Cài đặt tương tự cũng có trong `PyCharm <https://www.jetbrains.com/help/pycharm/debugger-python.html>`_.

.. note::

    | VS Code/pydev có thể cảnh báo bạn về:
    | ``sys.settrace() should not be used when the debugger is being used``
    | Nó có thể an toàn bị bỏ qua (và nếu bạn biết cách loại bỏ nó, hãy cho chúng tôi biết)

Bạn có thể thực thi run_single_user nhiều lần, như được hiển thị trong `debugging_advanced.py <https://github.com/locustio/locust/tree/master/examples/debugging_advanced.py>`_.


In ra thông tin giao tiếp HTTP (Print HTTP communication)
========================

Đôi khi có thể khó hiểu tại sao một yêu cầu HTTP thất bại trong Locust khi nó hoạt động từ trình duyệt thông thường/ứng dụng khác. Dưới đây là cách xem xét giao tiếp chi tiết:

Đối với ``HttpUser`` (`python-requests <https://python-requests.org>`_):

.. code-block:: python

    # đặt đoạn mã này ở đầu tệp locustfile của bạn (hoặc ngay trước yêu cầu bạn muốn theo dõi)
    import logging
    from http.client import HTTPConnection

    HTTPConnection.debuglevel = 1
    logging.basicConfig()
    logging.getLogger().setLevel(logging.DEBUG)
    requests_log = logging.getLogger("requests.packages.urllib3")
    requests_log.setLevel(logging.DEBUG)
    requests_log.propagate = True

Đối với ``FastHttpUser`` (`geventhttpclient <https://github.com/gwik/geventhttpclient/>`_):

.. code-block:: python

    import sys
    ...

    class MyUser(FastHttpUser):
        @task
        def t(self):
            self.client.get("http://example.com/", debug_stream=sys.stderr)

Ví dụ đầu ra (cho FastHttpUser):

.. code-block:: console

    REQUEST: http://example.com/
    GET / HTTP/1.1
    user-agent: python/gevent-http-client-1.5.3
    accept-encoding: gzip, deflate
    host: example.com

    RESPONSE: HTTP/1.1 200
    Content-Encoding: gzip
    Accept-Ranges: bytes
    Age: 460461
    Cache-Control: max-age=604800
    Content-Type: text/html; charset=UTF-8
    Date: Fri, 12 Aug 2022 09:20:07 GMT
    Etag: "3147526947+ident"
    Expires: Fri, 19 Aug 2022 09:20:07 GMT
    Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT
    Server: ECS (nyb/1D20)
    Vary: Accept-Encoding
    X-Cache: HIT
    Content-Length: 648

    <!doctype html>
    <html>
    <head>
    ...

Những cách tiếp cận này tất nhiên có thể được sử dụng khi thực hiện một thử nghiệm tải đầy đủ, nhưng bạn có thể nhận được rất nhiều đầu ra :)