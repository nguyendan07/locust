.. _extensions:

======================
Phần mở rộng của bên thứ ba (Third party extensions)
======================

Hỗ trợ kiểm tra tải cho các giao thức khác, báo cáo, v.v.
-------------------------------------------------------

-  `locust-plugins <https://github.com/SvenskaSpel/locust-plugins/>`__

   -  request logging & graphing
   -  các giao thức mới như websockets, selenium/webdriver, http users
      load html page resources
   -  readers (cách để đưa dữ liệu test vào tests của bạn)
   -  wait time (hàm chờ thời gian tùy chỉnh)
   -  debug (hỗ trợ chạy một user duy nhất trong trình gỡ lỗi)
   -  checks (thêm các tham số dòng lệnh để đặt mã thoát locust dựa
      trên requests/s, tỷ lệ lỗi và thời gian phản hồi trung bình)

Báo cáo traces OTEL cho các requests
------------------------------------

- `opentelemetry-demo repo <https://github.com/open-telemetry/opentelemetry-demo/tree/main/src/loadgenerator>`__

Tự động hóa các chạy phân tán qua SSH
--------------------------------------

-  `locust-swarm <https://github.com/SvenskaSpel/locust-swarm/>`__

Tự động dịch bản ghi trình duyệt (tệp HAR) sang tệp locustfile
-----------------------------------------------------------------------------

-  `har2locust <https://github.com/SvenskaSpel/har2locust>`__

Workers được viết bằng các ngôn ngữ khác ngoài Python
----------------------------------------------

Một master Locust và một worker Locust giao tiếp bằng cách trao đổi
`msgpack <http://msgpack.org/>`__ messages, được hỗ trợ bởi nhiều ngôn
ngữ. Vì vậy, bạn có thể viết các User tasks của mình bằng bất kỳ ngôn
ngữ nào bạn thích. Để tiện lợi, một số thư viện thực hiện công việc
như một worker runner. Chúng chạy các User tasks của bạn, và báo cáo
đến master đều đặn.

-  `Boomer <https://github.com/myzhan/boomer/>`__ - Go
-  `Locust4j <https://github.com/myzhan/locust4j>`__ - Java
