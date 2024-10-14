.. _increase-performance:

==============================================================
Tăng hiệu suất với HTTP client nhanh hơn (Increase performance with a faster HTTP client)
==============================================================

HTTP client mặc định của Locust sử dụng `python-requests <http://www.python-requests.org/>`_.
Nó cung cấp một API tốt mà nhiều nhà phát triển python quen thuộc, và được bảo trì rất tốt. Nhưng nếu bạn đang lên kế hoạch chạy các bài kiểm tra với tải cao và có phần cứng giới hạn để chạy Locust, đôi khi nó không hiệu quả đủ.

Vì vậy, Locust cũng đi kèm với :py:class:`FastHttpUser <locust.contrib.fasthttp.FastHttpUser>` sử dụng `geventhttpclient <https://github.com/gwik/geventhttpclient/>`_ thay thế.
Nó cung cấp một API rất tương tự và sử dụng ít thời gian CPU hơn, đôi khi tăng số lượng yêu cầu tối đa trên giây trên phần cứng cụ thể lên đến 5x-6x.

Không thể nói được bao nhiêu yêu cầu Locust có thể thực hiện trên phần cứng cụ thể của bạn, sử dụng kế hoạch kiểm tra cụ thể của bạn, vì vậy bạn cần thử nghiệm nó. Kiểm tra đầu ra console của Locust, nó sẽ ghi nhật ký cảnh báo nếu bị giới hạn bởi CPU.

Trong trường hợp *tốt nhất* (thực hiện các yêu cầu nhỏ trong một vòng lặp ``while True``) một quy trình Locust duy nhất (giới hạn một lõi CPU) có thể thực hiện khoảng **16000 yêu cầu mỗi giây sử dụng FastHttpUser, và 4000 sử dụng HttpUser** (đã kiểm tra trên M1 MacBook Pro 2021 và Python 3.11)

Sự cải thiện tương đối có thể lớn hơn với các tải trọng yêu cầu lớn hơn, nhưng cũng có thể nhỏ hơn nếu bài kiểm tra của bạn đang thực hiện các công việc tốn CPU không liên quan đến yêu cầu.

Tất nhiên, trong thực tế, bạn nên chạy :ref:`một quy trình Locust cho mỗi lõi CPU <running-distributed>`.

.. note::

    Miễn là CPU của máy tạo tải không bị quá tải, thời gian phản hồi của FastHttpUser sẽ gần như giống với của HttpUser. Nó không làm cho các yêu cầu riêng lẻ nhanh hơn.

Cách sử dụng FastHttpUser
===========================

Chỉ cần kế thừa FastHttpUser thay vì HttpUser::

    from locust import task, FastHttpUser
    
    class MyUser(FastHttpUser):
        @task
        def index(self):
            response = self.client.get("/")

Đồng thời (concurrency)
===========

Một phiên FastHttpUser/geventhttpclient có thể chạy các yêu cầu đồng thời, bạn chỉ cần khởi chạy greenlets cho mỗi yêu cầu::

    @task
    def t(self):
        def concurrent_request(url):
            self.client.get(url)

        pool = gevent.pool.Pool()
        urls = ["/url1", "/url2", "/url3"]
        for url in urls:
            pool.spawn(concurrent_request, url)
        pool.join()


.. note::

    FastHttpUser/geventhttpclient rất giống với HttpUser/python-requests, nhưng đôi khi có sự khác biệt tinh tế. Điều này đặc biệt đúng nếu bạn làm việc với các phần bên trong của thư viện khách, ví dụ: khi quản lý cookie thủ công.

.. _rest:

REST
====

FastHttpUser cung cấp một phương thức ``rest`` để kiểm tra các giao diện HTTP REST/JSON. Đây là một bọc cho ``self.client.request`` mà:

* Phân tích cú pháp JSON phản hồi thành một dict được gọi là ``js`` trong đối tượng phản hồi. Đánh dấu yêu cầu là thất bại nếu phản hồi không phải là JSON hợp lệ.
* Mặc định các tiêu đề ``Content-Type`` và ``Accept`` thành ``application/json``
* Đặt ``catch_response=True`` (vì vậy luôn sử dụng một :ref:`with-block <catch-response>`)
* Bắt bất kỳ ngoại lệ không được xử lý nào được ném trong with-block của bạn, đánh dấu mẫu là thất bại (thay vì thoát khỏi nhiệm vụ ngay lập tức mà không thậm chí kích hoạt sự kiện yêu cầu)

.. code-block:: python

    from locust import task, FastHttpUser

    class MyUser(FastHttpUser):
        @task
        def t(self):
            with self.rest("POST", "/", json={"foo": 1}) as resp:
                if resp.js is None:
                    pass # không cần làm gì, đã được đánh dấu là thất bại
                elif "bar" not in resp.js:
                    resp.failure(f"'bar' missing from response {resp.text}")
                elif resp.js["bar"] != 42:
                    resp.failure(f"'bar' had an unexpected value: {resp.js['bar']}")

Để xem một ví dụ hoàn chỉnh, hãy xem `rest.py <https://github.com/locustio/locust/blob/master/examples/rest.py>`_. Đó cũng cho thấy cách bạn có thể sử dụng kế thừa để cung cấp các hành vi cụ thể cho REST API của bạn mà là chung cho nhiều yêu cầu/kế hoạch kiểm tra.

.. note::

    Tính năng này mới và chi tiết của giao diện/hiện thực có thể thay đổi trong các phiên bản mới của Locust.


Xử lý kết nối (Connection Handling)
===================================

Mặc định, Người dùng sẽ sử dụng lại cùng một kết nối TCP/HTTP (trừ khi nó bị hỏng một cách nào đó). Để mô phỏng thêm thực tế các trình duyệt mới kết nối đến ứng dụng của bạn, kết nối này có thể được đóng thủ công.

.. code-block:: python

        @task
        def t(self):
            self.client.client.clientpool.close() # self.client.client không phải là một lỗi chính tả
            self.client.get("/")                  # Ở đây một kết nối mới sẽ được tạo


API
===


Lớp FastHttpUser
--------------------

.. autoclass:: locust.contrib.fasthttp.FastHttpUser
    :members: network_timeout, connection_timeout, max_redirects, max_retries, insecure, proxy_host, proxy_port, concurrency, client_pool, rest, rest_


Lớp FastHttpSession
---------------------

.. autoclass:: locust.contrib.fasthttp.FastHttpSession
    :members: request, get, post, delete, put, head, options, patch

.. autoclass:: locust.contrib.fasthttp.FastResponse
    :members: content, text, json, headers
