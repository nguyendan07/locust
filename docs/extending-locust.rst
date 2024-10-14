.. _extending_locust:

===========
Mở rộng Locust - Event hooks
===========

Locust đi kèm với một số event hooks mà có thể được sử dụng để mở rộng Locust theo nhiều cách khác nhau.

Ví dụ, đây là cách thiết lập một trình nghe sự kiện sẽ kích hoạt sau khi một yêu cầu được hoàn thành::

    from locust import events

    @events.request.add_listener
    def my_request_handler(request_type, name, response_time, response_length, response,
                           context, exception, start_time, url, **kwargs):
        if exception:
            print(f"Request to {name} failed with exception {exception}")
        else:
            print(f"Successfully made a request to: {name}")
            print(f"The response was {response.text}")

.. note::

    Trong ví dụ trên, đối số ``**kwargs`` sẽ trống, vì chúng tôi đang xử lý tất cả các đối số, nhưng nó ngăn mã từ việc bị hỏng nếu các đối số mới được thêm trong một phiên bản tương lai của Locust.

    Ngoài ra, hoàn toàn có thể triển khai một khách hàng không cung cấp tất cả các tham số cho sự kiện này.
    Ví dụ, các giao thức không phải HTTP có thể không có khái niệm về `url` hoặc đối tượng `response`.
    Loại bỏ bất kỳ trường bị thiếu nào từ định nghĩa hàm trình nghe của bạn hoặc sử dụng đối số mặc định.

Khi chạy locust ở chế độ phân tán, có thể hữu ích khi thực hiện một số thiết lập trên các nút worker trước khi chạy các bài kiểm tra của bạn.
Bạn có thể kiểm tra để đảm bảo rằng bạn không chạy trên nút master bằng cách kiểm tra loại của :py:attr:`runner <locust.env.Environment.runner>`::

    from locust import events
    from locust.runners import MasterRunner

    @events.test_start.add_listener
    def on_test_start(environment, **kwargs):
        if not isinstance(environment.runner, MasterRunner):
            print("Beginning test setup")
        else:
            print("Started test from Master node")

    @events.test_stop.add_listener
    def on_test_stop(environment, **kwargs):
        if not isinstance(environment.runner, MasterRunner):
            print("Cleaning up test data")
        else:
            print("Stopped test from Master node")

Bạn cũng có thể sử dụng các sự kiện `để thêm các đối số dòng lệnh tùy chỉnh <https://github.com/locustio/locust/tree/master/examples/add_command_line_argument.py>`_.

Để xem danh sách đầy đủ các sự kiện có sẵn, hãy xem :ref:`events`.

.. _request_context:

Context yêu cầu (Request context)
===============

Sự kiện :py:attr:`request <locust.event.Events.request>` có một tham số context cho phép bạn truyền dữ liệu `về` yêu cầu (như tên người dùng, thẻ v.v.). Nó có thể được đặt trực tiếp trong cuộc gọi đến phương thức yêu cầu hoặc ở cấp độ Người dùng, bằng cách ghi đè phương thức User.context().

Ngữ cảnh (context) từ phương thức yêu cầu (request method)::

    class MyUser(HttpUser):
        @task
        def t(self):
            self.client.post("/login", json={"username": "foo"})
            self.client.get("/other_request", context={"username": "foo"})

        @events.request.add_listener
        def on_request(context, **kwargs):
            if context:
                print(context["username"])

Ngữ cảnh (context) từ User instance::

    class MyUser(HttpUser):
        def context(self):
            return {"username": self.username}

        @task
        def t(self):
            self.username = "foo"
            self.client.post("/login", json={"username": self.username})

        @events.request.add_listener
        def on_request(context, **kwargs):
            print(context["username"])


Ngữ cảnh (context) từ giá trị trong phản hồi, sử dụng :ref:`catch_response <catch-response>`::

    with self.client.get("/", catch_response=True) as resp:
        resp.request_meta["context"]["requestId"] = resp.json()["requestId"]


.. note::

    Bối cảnh yêu cầu không thay đổi cách thống kê thông thường của Locust được tính toán. Các giải pháp ghi nhật ký/báo cáo như `locust.cloud <https://locust.cloud/>`_ sử dụng cơ chế trên để lưu bối cảnh vào cơ sở dữ liệu.

Thêm Web Routes
==================

Locust sử sụng Flask để phục vụ giao diện web và do đó rất dễ thêm các điểm cuối web vào giao diện web.
Bằng cách lắng nghe sự kiện :py:attr:`init <locust.event.Events.init>`, chúng ta có thể lấy tham chiếu
đến thể hiện ứng dụng Flask và sử dụng nó để thiết lập một route mới::

    from locust import events

    @events.init.add_listener
    def on_locust_init(environment, **kw):
        @environment.web_ui.app.route("/added_page")
        def my_added_page():
            return "Another page"

Bây giờ bạn có thể bắt đầu locust và duyệt đến http://127.0.0.1:8089/added_page. Lưu ý rằng nó không được tự động thêm vào dưới dạng một tab mới - bạn sẽ cần nhập URL trực tiếp.

Mở rộng Giao diện Web
================

Thay vì thêm các route web đơn giản, bạn có thể sử dụng `Flask Blueprints
<https://flask.palletsprojects.com/en/1.1.x/blueprints/>`_ và `templates
<https://flask.palletsprojects.com/en/1.1.x/tutorial/templates/>`_ để không chỉ thêm các route mà còn mở rộng
giao diện web để cho phép bạn hiển thị dữ liệu tùy chỉnh cùng với các thống kê Locust tích hợp. Điều này phức tạp hơn nhưng có thể tăng cường đáng kể tính hữu ích và tùy chỉnh của giao diện web.

Các ví dụ về cách mở rộng giao diện web có thể được tìm thấy
trong `thư mục ví dụ <https://github.com/locustio/locust/tree/master/examples>`_ của mã nguồn Locust.

*  ``extend_modern_web_ui.py``: Hiển thị một bảng với content-length cho mỗi cuộc gọi.

* ``web_ui_cache_stats.py``: Hiển thị Varnish Hit/Miss stats cho mỗi cuộc gọi. Điều này có thể dễ dàng mở rộng sang các CDN hoặc proxy cache khác và thu thập các thống kê cache khác như tuổi cache, kiểm soát, ...

 .. image:: images/extend_modern_web_ui_cache_stats.png


Thêm Xác thực vào Giao diện Web
===================================

Locust sử dụng `Flask-Login <https://pypi.org/project/Flask-Login/>`_ để xử lý xác thực. ``login_manager`` được
tiết lộ trên ``environment.web_ui.app``, cho phép linh hoạt cho bạn triển khai bất kỳ loại xác thực nào bạn muốn!

Để sử dụng xác thực tên người dùng/mật khẩu, đơn giản cung cấp một ``username_password_callback`` cho ``environment.web_ui.auth_args``.
Bạn chịu trách nhiệm định nghĩa route cho callback và triển khai xác thực.

Các nhà cung cấp xác thực cũng có thể được cấu hình để cho phép xác thực từ bên thứ ba như GitHub hoặc SSO.
Đơn giản cung cấp một danh sách các ``auth_providers`` mong muốn. Bạn có thể chỉ định ``label`` và ``icon`` cho hiển thị trên nút.
``callback_url`` sẽ là url mà nút chuyển hướng đến. Bạn sẽ chịu trách nhiệm định nghĩa route callback cũng như xác thực với bên thứ ba.

Dù bạn đang sử dụng xác thực tên người dùng/mật khẩu, một nhà cung cấp xác thực, hoặc cả hai, một ``user_loader`` cần được cung cấp
đến ``login_manager``. ``user_loader`` nên trả về ``None`` để từ chối xác thực hoặc trả về một đối tượng User khi
xác thực với ứng dụng nên được cấp quyền.

Một ví dụ đầy đủ có thể được xem `trong ví dụ về xác thực <https://github.com/locustio/locust/tree/master/examples/web_ui_auth.py>`_.


Chạy một greenlet nền (background greenlet)
=========================

Bởi vì một tệp locust "chỉ là mã", không có gì ngăn cản bạn khởi chạy greenlet riêng của mình để chạy song song với tải/người dùng thực sự của bạn.

Ví dụ, bạn có thể theo dõi tỷ lệ thất bại của bài kiểm tra của bạn và dừng chạy nếu nó vượt quá một ngưỡng:

.. code-block:: python

    import gevent
    from locust import events
    from locust.runners import STATE_STOPPING, STATE_STOPPED, STATE_CLEANUP, MasterRunner, LocalRunner

    def checker(environment):
        while not environment.runner.state in [STATE_STOPPING, STATE_STOPPED, STATE_CLEANUP]:
            time.sleep(1)
            if environment.runner.stats.total.fail_ratio > 0.2:
                print(f"fail ratio was {environment.runner.stats.total.fail_ratio}, quitting")
                environment.runner.quit()
                return


    @events.init.add_listener
    def on_locust_init(environment, **_kwargs):
        # không chạy cái này trên worker, chúng ta chỉ quan tâm đến các con số tổng hợp
        if isinstance(environment.runner, MasterRunner) or isinstance(environment.runner, LocalRunner):
            gevent.spawn(checker, environment)

.. _parametrizing-locustfiles:


Tham số hóa tệp locust (Parametrizing locustfiles)
=========================

Có hai cách chính để tham số hóa tệp locust của bạn.

Biến môi trường cơ bản
---------------------------

Giống như với bất kỳ chương trình nào khác, bạn có thể sử dụng biến môi trường:

Trên linux/mac:

.. code-block:: bash

    MY_FUNKY_VAR=42 locust ...

Trên windows:

.. code-block:: bash

    SET MY_FUNKY_VAR=42
    locust ...

... và sau đó truy cập chúng trong tệp locust của bạn.

.. code-block:: python

    import os
    print(os.environ['MY_FUNKY_VAR'])

.. _custom-arguments:

Đối số tùy chỉnh
----------------

Bạn có thể thêm các đối số dòng lệnh tùy chỉnh của riêng mình vào Locust, sử dụng :py:attr:`init_command_line_parser <locust.event.Events.init_command_line_parser>` Event. Các đối số tùy chỉnh cũng được hiển thị và có thể chỉnh sửa trong giao diện web. Nếu `choices` được chỉ định cho đối số, chúng sẽ được hiển thị dưới dạng một dropdown trong giao diện web.

.. literalinclude:: ../examples/add_command_line_argument.py
    :language: python

Khi chạy Locust :ref:`phân tán <running-distributed>`, các đối số tùy chỉnh được tự động chuyển tiếp cho các worker khi chạy được bắt đầu (nhưng không phải trước đó, vì vậy bạn không thể dựa vào các đối số được chuyển tiếp *trước* khi bài kiểm tra thực sự đã bắt đầu).


Quản lý dữ liệu kiểm tra (Test data management)
====================

Có một số cách để đưa dữ liệu kiểm tra vào các bài kiểm tra của bạn (sau tất cả, bài kiểm tra của bạn chỉ là một chương trình Python và nó có thể làm bất cứ điều gì Python có thể). Các sự kiện của Locust cho phép bạn kiểm soát chi tiết *khi* để lấy/dừng dữ liệu kiểm tra. Bạn có thể tìm thấy `một ví dụ chi tiết ở đây <https://github.com/locustio/locust/tree/master/examples/test_data_management.py>`_.


Ví dụ khác
=============

Xem `locust-plugins <https://github.com/SvenskaSpel/locust-plugins#listeners>`_
