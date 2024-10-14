.. _faq:

===
Câu hỏi thường gặp - FAQ
===

Làm sao để...

Giải quyết lỗi xảy ra trong quá trình tải (lỗi 5xx, Kết nối bị hủy, Kết nối bị đặt bởi đối phương, v.v)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   Kiểm tra nhật ký máy chủ của bạn. Nếu nó hoạt động ở tải thấp thì nó chắc chắn không phải là vấn đề của Locust, mà là vấn đề của hệ thống bạn đang kiểm tra.

Xóa cookies, để lần lặp nhiệm vụ tiếp theo giống như một người dùng mới đối với hệ thống của tôi
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   Gọi ``self.client.cookies.clear()`` ở cuối nhiệm vụ của bạn.

Tôi muốn kiểm soát tiêu đề hoặc các thông tin khác về các yêu cầu HTTP của mình
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   Client của Locust trong ``HttpUser`` kế thừa từ `requests <https://requests.readthedocs.io/en/master/>`__ và hầu hết các tham số và phương thức cho yêu cầu nên hoạt động với Locust. Hãy xem tài liệu và câu trả lời trên Stack Overflow cho requests trước và sau đó hỏi trên Stack Overflow để được trợ giúp cụ thể về Locust nếu cần.

Xác thực cơ bản (Authorization header) không hoạt động sau khi chuyển hướng
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   `requests <https://requests.readthedocs.io/en/master/>`__ có một cơ chế bảo mật loại bỏ tiêu đề xác thực khi chuyển đổi miền. Điều này có thể xảy ra khi kiểm tra một SSO, thường là trên một miền khác và sử dụng nhiều chuyển hướng (30x).

   Vì ``allow_redirects=True`` là hành vi mặc định của ``request`` nên bạn phải tắt nó,
   xử lý chuyển hướng thủ công và tiêm lại tiêu đề xác thực, ví dụ::

.. code-block:: python

    response = self.client.post(
             allow_redirects=False,
             url=...)

    while "location" in response.headers:
        response = self.client.get(
        allow_redirects=False,
        url=response.headers['location'],
        headers={
          'Authorization': 'XXX'
        }
    )


Tạo một tệp Locust dựa trên một phiên trình duyệt được ghi lại
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   Thử sử dụng `Transformer <https://transformer.readthedocs.io/>`__

Làm cách nào để chạy một container Docker của Locust trong Windows Subsystem for Linux (người dùng Windows 10)?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   Nếu bạn sử dụng WSL trên máy tính Windows, thì bạn cần một bước bổ sung
   so với lệnh `“docker run …”
   command <https://docs.locust.io/en/stable/running-locust-docker.html>`__:
   sao chép locusttest1.py của bạn vào một thư mục trong một đường dẫn Windows trên WSL của bạn
   và gắn thư mục đó thay vì thư mục WSL bình thường của bạn:

::

   $ mkdir /c/Users/[YOUR_Windows_USER]/Documents/Locust/
   $ cp ~/path/to/locusttest1.py /c/Users/[YOUR_Windows_USER]/Documents/Locust/
   $ docker run -p 8089:8089 -v /c/Users/[YOUR_Windows_USER]/Documents/Locust/:/mnt/locust locustio/locust:1.3.1 -f /mnt/locust/locusttest1.py

Làm cách nào để chạy locust trên điểm cuối tùy chỉnh
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   Tiền tố điểm cuối vào tất cả các định nghĩa ``@app.route`` trong tệp ``locust/web.py`` và cũng thay đổi như sau (nơi ``/locust`` là điểm cuối mới)

``app = Flask(__name__, static_url_path='/locust')``

   Thay đổi các mục vị trí nội dung tĩnh trong tệp ``locust/templates/index.html``.

Ví dụ:
``<link rel="shortcut icon" href="{{ url_for('static', filename='img/favicon.ico') }}" type="image/x-icon"/>``

Giao diện web Locust không hiển thị các nhiệm vụ của tôi đang chạy, nó nói 0 RPS
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Locust chỉ biết những gì bạn đang làm khi bạn nói cho nó. Có `Event Hooks <https://docs.locust.io/en/stable/api.html#events>`__ mà bạn sử dụng để nói cho Locust điều gì đang xảy ra trong mã của bạn. Nếu bạn sử dụng ``HttpUser`` của Locust và sau đó sử dụng ``self.client`` để thực hiện cuộc gọi http, các sự kiện đúng thường được kích hoạt tự động cho bạn, giảm công việc cho bạn trừ khi bạn muốn ghi đè lên các sự kiện mặc định.

Nếu bạn sử dụng ``User`` thông thường hoặc sử dụng ``HttpUser`` và bạn không sử dụng ``self.client`` để thực hiện cuộc gọi http, Locust sẽ không kích hoạt sự kiện cho bạn. Bạn sẽ phải kích hoạt sự kiện mình. Xem `tài liệu Locust <https://docs.locust.io/en/stable/testing-other-systems.html>`__ để biết ví dụ.

Báo cáo HTML được điền đầy yêu cầu thất bại cho các thử nghiệm chạy lâu
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

https://github.com/locustio/locust/issues/2328

Các câu hỏi và vấn đề khác
~~~~~~~~~~~~~~~~~~~~~~~~~~

`Kiểm tra danh sách vấn đề (nhiều câu hỏi/sự hiểu lầm được đưa ra
dưới dạng vấn đề) <https://github.com/locustio/locust/issues?q=is%3Aissue%20>`__

Thêm những điều bạn đã gặp phải và giải quyết ở đây! Bất kỳ ai có tài khoản GitHub
có thể đóng góp!

Nếu bạn có câu hỏi về Locust mà không được trả lời ở đây, vui lòng
kiểm tra
`StackOverflow <https://stackoverflow.com/questions/tagged/locust>`__,
hoặc đăng câu hỏi của bạn ở đó. Wiki này không dành cho việc hỏi câu hỏi mà
dành cho việc trả lời chúng :)
