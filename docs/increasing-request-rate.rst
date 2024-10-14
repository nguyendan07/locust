.. _increaserr:

===========================
Tăng tốc độ yêu cầu (Increasing the request rate)
===========================

Tăng số lượng yêu cầu mỗi giây bằng cách kết hợp các bước sau:

#. Tăng số lượng người dùng. Để tận dụng hệ thống mục tiêu của bạn, bạn có thể cần nhiều người dùng đồng thời, đặc biệt nếu mỗi yêu cầu mất thời gian lâu để hoàn thành.

#. Nếu thời gian phản hồi cao và/hoặc tăng khi số lượng người dùng tăng, thì bạn có thể đã bão hòa hệ thống bạn đang kiểm tra và cần phải tìm hiểu lý do tại sao. Điều này không phải là vấn đề của Locust, nhưng đây là một số điều bạn có thể muốn kiểm tra:

    -  sử dụng tài nguyên (ví dụ: CPU, bộ nhớ & mạng. Kiểm tra các chỉ số này ở phía Locust)
    -  cấu hình (ví dụ: số luồng tối đa cho máy chủ web của bạn)
    -  thời gian phản hồi phía sau (ví dụ: DB)
    -  hiệu suất DNS phía máy khách/bảo vệ tràn (Locust thường sẽ thực hiện ít nhất một yêu cầu DNS cho mỗi người dùng)

#. Nếu Locust in ra cảnh báo về việc sử dụng CPU cao (``WARNING/root: CPU usage above 90%! ...``) hãy thử các bước sau:

    -  Chạy Locust `phân tán <https://docs.locust.io/en/stable/running-locust-distributed.html>`__ để tận dụng nhiều lõi & nhiều máy
    -  Thử chuyển sang `FastHttpUser <https://docs.locust.io/en/stable/increase-performance.html#increase-performance>`__ để giảm việc sử dụng CPU
    -  Kiểm tra xem có vòng lặp lạ/không giới hạn nào trong mã của bạn không

#. Nếu bạn đang sử dụng một client tùy chỉnh (không phải HttpUser hoặc FastHttpUser), hãy chắc chắn rằng bất kỳ thư viện client nào bạn đang sử dụng đều thân thiện với gevent, nếu không nó sẽ chặn toàn bộ quá trình Python (về cơ bản giới hạn bạn chỉ có một người dùng cho mỗi worker)

.. note::

    Tốc độ hatch/khởi động không thay đổi tải cao nhất, nó chỉ thay đổi cách bạn đến đó.