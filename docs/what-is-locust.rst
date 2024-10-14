===============================
##  Locust là gì?
===============================

Locust là một công cụ kiểm tra hiệu suất/tải nguồn mở cho HTTP và các giao thức khác. Cách tiếp cận thân thiện với nhà phát triển cho phép bạn định nghĩa các bài kiểm tra của mình bằng mã Python thông thường.

Các bài kiểm tra Locust có thể được chạy từ dòng lệnh hoặc sử dụng giao diện web của nó. Lưu lượng, thời gian phản hồi và lỗi có thể được xem theo thời gian thực và/hoặc xuất ra để phân tích sau này.

Bạn có thể nhập các thư viện Python thông thường vào các bài kiểm tra của mình, và với kiến ​​trúc có thể cắm vào của Locust, nó có thể được mở rộng vô hạn. Không giống như khi sử dụng hầu hết các công cụ khác, thiết kế bài kiểm tra của bạn sẽ không bao giờ bị giới hạn bởi giao diện người dùng đồ họa hoặc ngôn ngữ cụ thể của miền.

Để bắt đầu sử dụng Locust, hãy truy cập: :ref:`installation`

## Tính năng
========

.. raw:: html

    <div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto;">
        <iframe src="https://www.youtube.com/embed/Ok4x2LIbEEY?start=163&end=1477;" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
    </div><br/>
    
* **Viết các kịch bản thử nghiệm bằng Python thuần túy**

  Nếu bạn muốn người dùng của mình lặp lại, thực hiện một số hành vi có điều kiện hoặc thực hiện một số tính toán, bạn chỉ cần sử dụng các cấu trúc lập trình thông thường được cung cấp bởi Python.
  Locust chạy mỗi người dùng bên trong greenlet riêng của nó (một quy trình/coroutine nhẹ). Điều này cho phép bạn viết các bài kiểm tra của mình như mã Python bình thường (chặn) thay vì phải sử dụng các hàm gọi lại hoặc một số cơ chế khác.
  Bởi vì các kịch bản của bạn là "chỉ Python" nên bạn có thể sử dụng IDE thông thường của mình và kiểm soát phiên bản các bài kiểm tra của mình như mã thông thường (trái ngược với một số công cụ khác sử dụng các định dạng XML hoặc nhị phân)

* **Phân tán và có thể mở rộng - hỗ trợ hàng trăm nghìn người dùng đồng thời**

  Locust giúp bạn dễ dàng chạy các bài kiểm tra tải được phân tán trên nhiều máy.
  Nó dựa trên sự kiện (sử dụng `gevent <http://www.gevent.org/>`_), điều này cho phép một quy trình duy nhất xử lý hàng nghìn người dùng đồng thời.
  Mặc dù có thể có các công cụ khác có khả năng thực hiện nhiều yêu cầu hơn mỗi giây trên phần cứng nhất định, nhưng chi phí thấp của mỗi người dùng Locust khiến nó rất phù hợp để kiểm tra khối lượng công việc đồng thời cao.

* **Giao diện web**

  Locust có giao diện web thân thiện với người dùng hiển thị tiến trình của bài kiểm tra của bạn theo thời gian thực. Bạn thậm chí có thể thay đổi tải trong khi bài kiểm tra đang chạy. Nó cũng có thể được chạy mà không cần giao diện người dùng, giúp bạn dễ dàng sử dụng để kiểm tra CI/CD.

* **Có thể kiểm tra bất kỳ hệ thống nào**

  Mặc dù Locust chủ yếu hoạt động với các trang web/dịch vụ, nhưng nó có thể được sử dụng để kiểm tra hầu hết mọi hệ thống hoặc giao thức. Chỉ cần :ref:`viết một client <testing-other-systems>` 
  cho những gì bạn muốn kiểm tra, hoặc `khám phá một số được tạo bởi cộng đồng <https://github.com/SvenskaSpel/locust-plugins#users>`_.

* **Có thể hack**

  Locust nhỏ và rất linh hoạt và chúng tôi dự định sẽ giữ nguyên như vậy. Nếu bạn muốn `gửi dữ liệu báo cáo đến cơ sở dữ liệu & hệ thống vẽ biểu đồ bạn thích <https://github.com/SvenskaSpel/locust-plugins/tree/master/locust_plugins/dashboards>`_, gói các cuộc gọi đến API REST để xử lý các chi tiết cụ thể của hệ thống của bạn hoặc chạy một :ref:`mô hình tải tùy chỉnh hoàn toàn <custom-load-shape>`, không có gì có thể ngăn bạn!

## Tên & bối cảnh
=================

Locust ra đời từ sự thất vọng với các giải pháp hiện có. Không có công cụ kiểm tra tải nào hiện có được trang bị tốt để tạo ra tải thực tế 
chống lại một trang web động, nơi hầu hết các trang có nội dung khác nhau cho các người dùng khác nhau. Các công cụ hiện có sử dụng giao diện cồng kềnh hoặc 
các tệp cấu hình dài dòng để khai báo các bài kiểm tra. Trong Locust, chúng tôi đã áp dụng một cách tiếp cận khác. Thay vì các định dạng cấu hình hoặc giao diện người dùng 
bạn sẽ nhận được một khung Python cho phép bạn định nghĩa hành vi của người dùng bằng mã Python.

Locust lấy tên từ loài `chuồn chuồn <https://en.wikipedia.org/wiki/Locust>`_, nổi tiếng với hành vi bầy đàn.

:ref:`history`

## Tác giả
=======

- Jonatan Heyman (`@heyman <https://github.com/heyman>`_)
- Lars Holmberg (`@cyberw <https://github.com/cyberw>`_)
- Andrew Baldwin (`@andrewbaldwin44 <https://github.com/andrewbaldwin44>`_)

Rất cảm ơn các `người đóng góp <https://github.com/locustio/locust/graphs/contributors>`_ tuyệt vời khác của chúng tôi!


## Giấy phép
=======

Giấy phép nguồn mở theo giấy phép MIT (xem tệp LICENSE để biết chi tiết).
