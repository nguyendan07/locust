:orphan:
.. _history:

===============================
The history of Locust
===============================

Locust được tạo ra vì chúng tôi đã chán ngấy với các giải pháp hiện có. Không có giải pháp nào
giải quyết vấn đề đúng cách và theo tôi, chúng thiếu điểm. Chúng tôi đã thử cả Apache JMeter và Tsung.
Cả hai công cụ đều khá tốt để sử dụng; chúng tôi đã sử dụng cái đầu tiên nhiều lần để đánh giá các vấn đề
tại công việc. JMeter đi kèm với giao diện người dùng, mà bạn có thể nghĩ trong một giây là một điều tốt.
Nhưng bạn sớm nhận ra rằng nó là một PITA để "lập trình" các kịch bản kiểm tra của mình thông qua một
giao diện nhấp chuột. Thứ hai, JMeter bị ràng buộc bởi luồng. Điều này có nghĩa là đối với mỗi người dùng
bạn muốn mô phỏng, bạn cần một luồng riêng biệt. Không cần phải nói, đánh giá hàng nghìn người dùng trên
một máy duy nhất đơn giản không khả thi.

Tsung, ngược lại, không có các vấn đề luồng này vì nó được viết bằng Erlang. Nó có thể sử dụng các
tiến trình nhẹ được cung cấp bởi BEAM chính và mở rộng hạnh phúc lên. Nhưng khi đến việc xác định
các kịch bản kiểm tra, Tsung bị hạn chế như JMeter. Nó cung cấp một DSL dựa trên XML để xác định
cách người dùng phải hành xử khi kiểm tra. Tôi đoán bạn có thể tưởng tượng sự kinh hoàng của "lập trình" này.
Hiển thị bất kỳ loại biểu đồ hoặc báo cáo nào khi hoàn thành yêu cầu bạn phải xử lý sau khi tạo ra
các tệp nhật ký từ kiểm tra. Chỉ sau đó bạn mới có thể hiểu được cách kiểm tra đã diễn ra.

Dù sao, chúng tôi đã cố gắng giải quyết các vấn đề này khi tạo ra Locust. Hy vọng không có bất kỳ
vấn đề nào ở trên điểm đau này nên tồn tại.

Tôi đoán bạn có thể nói rằng chúng tôi thực sự chỉ đang cố gắng giải quyết vấn đề của chính mình ở đây.
Chúng tôi hy vọng người khác sẽ thấy nó hữu ích như chúng tôi.


Locust was created because we were fed up with existing solutions. None of them are solving the 
right problem and to me, they are missing the point. We've tried both Apache JMeter and Tsung. 
Both tools are quite OK to use; we've used the former many times benchmarking stuff at work.
JMeter comes with a UI, which you might think for a second is a good thing. But you soon realize it's
a PITA to "code" your testing scenarios through some point-and-click interface. Secondly, JMeter 
is thread-bound. This means for every user you want to simulate, you need a separate thread. 
Needless to say, benchmarking thousands of users on a single machine just isn't feasible.

Tsung, on the other hand, does not have these thread issues as it's written in Erlang. It can make 
use of the light-weight processes offered by BEAM itself and happily scale up. But when it comes to 
defining the test scenarios, Tsung is as limited as JMeter. It offers an XML-based DSL to define how 
a user should behave when testing. I guess you can imagine the horror of "coding" this. Displaying 
any sorts of graphs or reports when completed requires you to post-process the log files generated from
the test. Only then can you get an understanding of how the test went.

Anyway, we've tried to address these issues when creating Locust. Hopefully none of the above 
pain points should exist.

I guess you could say we're really just trying to scratch our own itch here. We hope others will 
find it as useful as we do.

- `Jonatan Heyman <http://heyman.info>`_ (`@jonatanheyman <https://twitter.com/jonatanheyman>`_ on Twitter)
