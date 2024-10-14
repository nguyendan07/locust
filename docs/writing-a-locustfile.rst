.. _writing-a-locustfile:

======================
Viết một locustfile
======================

Bây giờ, hãy cùng xem xét một ví dụ hoàn chỉnh/thực tế hơn về cách các bài kiểm tra của bạn có thể trông như thế nào:

.. code-block:: python

    import time
    from locust import HttpUser, task, between

    class QuickstartUser(HttpUser):
        wait_time = between(1, 5)

        @task
        def hello_world(self):
            self.client.get("/hello")
            self.client.get("/world")

        @task(3)
        def view_items(self):
            for item_id in range(10):
                self.client.get(f"/item?id={item_id}", name="/item")
                time.sleep(1)

        def on_start(self):
            self.client.post("/login", json={"username":"foo", "password":"bar"})


.. rubric:: Let's break it down

.. code-block:: python

    import time
    from locust import HttpUser, task, between

Một locustfile chỉ là một mô-đun Python thông thường, nó có thể nhập mã từ các tệp hoặc gói khác.

.. code-block:: python

    class QuickstartUser(HttpUser):

Ở đây, chúng ta định nghĩa một lớp cho các user mà chúng ta sẽ mô phỏng. Lớp này kế thừa từ :py:class:`HttpUser <locust.HttpUser>`, nó cung cấp mỗi user một thuộc tính ``client``, là một phiên bản của :py:class:`HttpSession <locust.clients.HttpSession>`, mà có thể được sử dụng để thực hiện các yêu cầu HTTP đến hệ thống mục tiêu mà chúng ta muốn kiểm tra tải. Khi một bài kiểm tra bắt đầu, Locust sẽ tạo một phiên bản của lớp này cho mỗi user mà nó mô phỏng, và mỗi user này sẽ bắt đầu chạy trong một green gevent thread của riêng mình.

Để một tệp được coi là một locustfile hợp lệ, nó phải chứa ít nhất một lớp kế thừa từ :py:class:`User <locust.User>`.

.. code-block:: python

    wait_time = between(1, 5)

Lớp của chúng ta định nghĩa một ``wait_time`` sẽ khiến cho các user mô phỏng phải chờ giữa 1 và 5 giây sau mỗi nhiệm vụ (xem bên dưới) được thực thi. Để biết thêm thông tin, xem :ref:`wait-time`.

.. code-block:: python

    @task
    def hello_world(self):
        self.client.get("/hello")
        self.client.get("/world")

Các phương thức được trang trí bằng ``@task`` là cốt lõi của tệp locust của bạn. Đối với mỗi user đang chạy, Locust tạo ra một `greenlet <https://greenlet.readthedocs.io/en/stable/greenlet.html>`_ (một coroutine hoặc "micro-thread"), sẽ gọi các phương thức đó. Code trong một nhiệm vụ được thực thi tuần tự (nó chỉ là code Python thông thường), vì vậy ``/world`` sẽ không được gọi cho đến khi phản hồi từ ``/hello`` đã được nhận.

.. code-block:: python

    @task
    def hello_world(self):
        ...
    
    @task(3)
    def view_items(self):
        ...

Chúng ta đã định nghĩa hai nhiệm vụ bằng cách trang trí hai phương thức với ``@task``, một trong số đó đã được đặt trọng số cao hơn (3). Khi ``QuickstartUser`` của chúng ta chạy, nó sẽ chọn một trong hai nhiệm vụ đã định nghĩa - trong trường hợp này là ``hello_world`` hoặc ``view_items`` - và thực thi nó. Các nhiệm vụ được chọn ngẫu nhiên, nhưng bạn có thể đặt trọng số khác nhau. Cấu hình trên sẽ khiến cho Locust có khả năng chọn ``view_items`` gấp ba lần so với ``hello_world``. Khi một nhiệm vụ đã thực thi xong, user sẽ ngủ trong thời gian chờ được chỉ định (trong trường hợp này giữa 1 và 5 giây). Sau đó, nó sẽ chọn một nhiệm vụ mới.

Lưu ý rằng chỉ các phương thức được trang trí bằng ``@task`` sẽ được chọn, vì vậy bạn có thể tự định nghĩa các phương thức trợ giúp nội bộ theo cách bạn muốn.

.. code-block:: python

    self.client.get("/hello")

Thuộc tính ``self.client`` giúp bạn thực hiện các cuộc gọi HTTP sẽ được ghi lại bởi Locust. Để biết thêm thông tin về cách thực hiện các cuộc gọi khác, xác minh phản hồi, v.v., xem :ref:`Sử dụng HTTP Client <writing-a-locustfile.html#client-attribute-httpsession>`_.

.. note::

    HttpUser không phải là một trình duyệt thực sự, do đó nó sẽ không phân tích cú pháp phản hồi HTML để tải tài nguyên hoặc hiển thị trang. Tuy nhiên, nó sẽ theo dõi cookie.

.. code-block:: python

    @task(3)
    def view_items(self):
        for item_id in range(10):
            self.client.get(f"/item?id={item_id}", name="/item")
            time.sleep(1)

Trong tác vụ ``view_items`` chúng ta tải 10 URL khác nhau bằng cách sử dụng một tham số truy vấn biến.
Để không có 10 mục riêng lẻ trong thống kê của Locust - vì thống kê được nhóm theo URL - chúng ta sử dụng
:ref:`tham số name <name-parameter>` để nhóm tất cả các yêu cầu đó dưới một mục có tên là ``"/item"``.

.. code-block:: python

    def on_start(self):
        self.client.post("/login", json={"username":"foo", "password":"bar"})

Ngoài ra, chúng ta đã khai báo một phương thức ``on_start``. Một phương thức có tên này sẽ được gọi cho mỗi user được mô phỏng
khi chúng bắt đầu chạy. Để biết thêm thông tin, hãy xem :ref:`on-start-on-stop`.

Tạo locustfile tự động
============================

Bạn có thể sử dụng `har2locust <<https://github.com/SvenskaSpel/har2locust>`_ để tạo locustfiles dựa trên một bản ghi trình duyệt (tệp HAR).

Nó đặc biệt hữu ích cho người mới bắt đầu không quen viết locustfile của riêng mình, nhưng cũng rất linh hoạt cho các trường hợp sử dụng nâng cao hơn.

.. note::

    har2locust vẫn đang trong phiên bản beta. Nó có thể không luôn tạo ra các tệp locust đúng, và giao diện của nó có thể thay đổi giữa các phiên bản.

Lớp User
==========

Một lớp User đại diện cho một loại người dùng/kịch bản cho hệ thống của bạn. Khi bạn chạy một bài kiểm tra, bạn chỉ định số lượng người dùng đồng thời bạn muốn mô phỏng và Locust sẽ tạo một phiên bản cho mỗi người dùng. Bạn có thể thêm bất kỳ thuộc tính nào bạn muốn vào các lớp/phiên bản này, nhưng có một số thuộc tính có ý nghĩa đặc biệt với Locust:

.. _wait-time:

Thuộc tính wait_time
-------------------

Phương thức :py:attr:`wait_time <locust.User.wait_time>` của người dùng giúp bạn dễ dàng giới thiệu độ trễ sau mỗi thực thi nhiệm vụ. Nếu không có `wait_time` được chỉ định, nhiệm vụ tiếp theo sẽ được thực thi ngay sau khi một nhiệm vụ kết thúc.

* :py:attr:`constant <locust.wait_time.constant>` cho một khoảng thời gian cố định

* :py:attr:`between <locust.wait_time.between>` cho một khoảng thời gian ngẫu nhiên giữa một giá trị tối thiểu và tối đa

Ví dụ, để khiến mỗi người dùng phải chờ giữa 0,5 và 10 giây sau mỗi thực thi nhiệm vụ:

.. code-block:: python

    from locust import User, task, between

    class MyUser(User):
        @task
        def my_task(self):
            print("executing my_task")

        wait_time = between(0.5, 10)

* :py:attr:`constant_throughput <locust.wait_time.constant_throughput>` cho một thời gian thích ứng đảm bảo nhiệm vụ chạy (tối đa) X lần mỗi giây.

* :py:attr:`constant_pacing <locust.wait_time.constant_pacing>` cho một thời gian thích ứng đảm bảo nhiệm vụ chạy (tối đa) một lần mỗi X giây (đây là nghịch đảo toán học của `constant_throughput`).

.. note::

    Thời gian chờ chỉ có thể hạn chế tốc độ thực thi, không thể khởi chạy người dùng mới để đạt mục tiêu. Ví dụ, nếu bạn muốn Locust chạy 500 lần thực thi nhiệm vụ mỗi giây ở tải cao nhất, bạn có thể sử dụng `wait_time = constant_throughput(0.1)` và số lượng người dùng là 5000.

    Thời gian chờ chỉ có thể hạn chế lưu lượng, không khởi chạy User mới để đạt được mục tiêu. Vì vậy, trong ví dụ của chúng ta, lưu lượng sẽ nhỏ hơn 500 nếu thời gian cho lần lặp tác vụ vượt quá 10 giây.
    
    Thời gian chờ được áp dụng *after* thực thi nhiệm vụ, vì vậy nếu bạn có một tỷ lệ khởi chạy cao/tăng dần bạn có thể vượt quá mục tiêu của mình trong quá trình tăng dần.

    Thời gian chờ áp dụng cho *tasks*, không phải yêu cầu. Ví dụ, nếu bạn chỉ định `wait_time = constant_throughput(2)` và thực hiện hai yêu cầu trong các nhiệm vụ của bạn, tỷ lệ yêu cầu/RPS của bạn sẽ là 4 cho mỗi User.

Bạn cũng có thể khai báo phương thức wait_time của riêng mình trực tiếp trên lớp của mình.
Ví dụ, lớp User dưới đây sẽ ngủ một giây, sau đó hai giây, sau đó ba giây, v.v.

.. code-block:: python

    class MyUser(User):
        last_wait_time = 0

        def wait_time(self):
            self.last_wait_time += 1
            return self.last_wait_time

        ...


Thuộc tính weight và fixed_count
---------------------------------

Nếu có nhiều hơn một lớp người dùng tồn tại trong tệp, và không có lớp người dùng nào được chỉ định trên dòng lệnh,
Locust sẽ tạo ra một số lượng bằng nhau của mỗi lớp người dùng. Bạn cũng có thể chỉ định lớp người dùng nào sẽ sử dụng từ cùng một locustfile bằng cách truyền chúng như đối số dòng lệnh:

.. code-block:: console

    $ locust -f locust_file.py WebUser MobileUser

Nếu bạn muốn mô phỏng nhiều người dùng của một loại hơn một loại khác, bạn có thể đặt một thuộc tính trọng số trên các lớp đó.
Mã dưới đây sẽ khiến Locust tạo ra 3 lần nhiều WebUsers so với MobileUsers:

.. code-block:: python

    class WebUser(User):
        weight = 3
        ...

    class MobileUser(User):
        weight = 1
        ...

Ngoài ra, bạn có thể đặt thuộc tính :py:attr:`fixed_count <locust.User.fixed_count>`. Trong trường hợp này, thuộc tính trọng số sẽ bị bỏ qua và chỉ có một số lượng cụ thể người dùng sẽ được tạo ra. Những người dùng này sẽ được tạo ra trước bất kỳ người dùng nào khác có trọng số. Trong ví dụ dưới đây, chỉ một phiên bản của AdminUser sẽ được tạo ra, để thực hiện một số công việc cụ thể với kiểm soát số lượng yêu cầu chính xác độc lập với số lượng người dùng tổng.

.. code-block:: python

    class AdminUser(User):
        wait_time = constant(600)
        fixed_count = 1
        
        @task
        def restart_app(self):
            ...

    class WebUser(User):
        ...


Thuộc tính host
--------------

Thuộc tính host là một tiền tố URL (ví dụ: ``https://google.com``) cho máy chủ bạn muốn kiểm tra. Nó sẽ tự động được thêm vào các yêu cầu, vì vậy bạn có thể thực hiện ``self.client.get("/")`` ví dụ.

Bạn có thể ghi đè giá trị này trong giao diện web của Locust hoặc trên dòng lệnh, sử dụng tùy chọn :code:`--host`.

Thuộc tính tasks
----------------

Một lớp User có thể có các nhiệm vụ được khai báo dưới dạng phương thức bằng cách sử dụng trang trí :py:func:`@task <locust.task>`, nhưng bạn cũng có thể
xác định các nhiệm vụ bằng cách sử dụng thuộc tính *tasks*, mà được mô tả chi tiết hơn :ref:`dưới đây <tasks-attribute>`.

Thuộc tính môi trường
---------------------

Một tham chiếu đến :py:attr:`môi trường <locust.env.Environment>` mà người dùng đang chạy. Sử dụng nó để tương tác với môi trường,
hoặc :py:attr:`runner <locust.runners.Runner>` mà nó chứa. Ví dụ, để dừng runner từ một phương thức nhiệm vụ:

.. code-block:: python

    self.environment.runner.quit()

Nếu chạy trên một phiên bản Locust độc lập, điều này sẽ dừng toàn bộ bài kiểm tra. Nếu chạy trên nút worker, nó sẽ dừng nút cụ thể đó.

.. _on-start-on-stop:

Phương thức on_start và on_stop
-------------------------------

Người dùng (và :ref:`TaskSets <tasksets>`) có thể khai báo một phương thức :py:meth:`on_start <locust.User.on_start>` và/hoặc
:py:meth:`on_stop <locust.User.on_stop>`. Một User sẽ gọi phương thức
:py:meth:`on_start <locust.User.on_start>` của mình khi nó bắt đầu chạy, và phương thức
:py:meth:`on_stop <locust.User.on_stop>` của nó khi nó dừng chạy. Đối với TaskSet, phương thức
:py:meth:`on_start <locust.TaskSet.on_start>` được gọi khi một người dùng mô phỏng bắt đầu thực thi
TaskSet đó, và :py:meth:`on_stop <locust.TaskSet.on_stop>` được gọi khi người dùng mô phỏng dừng
thực thi TaskSet đó (khi :py:meth:`interrupt() <locust.TaskSet.interrupt>` được gọi, hoặc người
dùng bị giết).

Nhiệm vụ
========

Khi một bài kiểm tra bắt đầu, một phiên bản của một lớp User sẽ được tạo ra cho mỗi người dùng mô phỏng và họ sẽ bắt đầu chạy trong greenlet riêng của mình. Khi những người dùng này chạy, họ chọn các nhiệm vụ mà họ thực thi, ngủ một thời gian, và sau đó chọn một nhiệm vụ mới và v.v.

@task decorator
---------------

Cách dễ nhất để thêm một nhiệm vụ cho một User là bằng cách sử dụng :py:meth:`task <locust.task>` decorator.

.. code-block:: python

    from locust import User, task, constant

    class MyUser(User):
        wait_time = constant(1)

        @task
        def my_task(self):
            print("User instance (%r) executing my_task" % self)

**@task** nhận một đối số weight tùy chọn có thể được sử dụng để chỉ định tỷ lệ thực thi của tác vụ. Trong
ví dụ dưới đây, *task2* sẽ có khả năng được chọn gấp đôi so với *task1*:

.. code-block:: python

    from locust import User, task, between

    class MyUser(User):
        wait_time = between(5, 15)

        @task(3)
        def task1(self):
            pass

        @task(6)
        def task2(self):
            pass


.. _tasks-attribute:

Thuộc tính tasks
---------------

Một cách khác để xác định các tác vụ của một User là đặt thuộc tính :py:attr:`tasks <locust.User.tasks>`.

Thuộc tính *tasks* là một danh sách các Tasks, hoặc một từ điển *<Task : int>*, trong đó Task là một
callable Python hoặc một lớp :ref:`TaskSet <tasksets>`. Nếu nhiệm vụ là một hàm Python bình thường, chúng
nhận một đối số duy nhất là thể hiện User đang thực thi nhiệm vụ.

Dưới đây là một ví dụ về một nhiệm vụ User được định nghĩa như một hàm Python bình thường:

.. code-block:: python

    from locust import User, constant

    def my_task(user):
        pass

    class MyUser(User):
        tasks = [my_task]
        wait_time = constant(1)


Nếu thuộc tính tasks được chỉ định dưới dạng danh sách, mỗi khi một nhiệm vụ được thực thi, nó sẽ được chọn ngẫu nhiên từ thuộc tính *tasks*. Tuy nhiên, nếu *tasks* là một từ điển - với các callable làm khóa và số nguyên làm giá trị - nhiệm vụ sẽ được chọn ngẫu nhiên nhưng với tỷ lệ số nguyên. Vì vậy, với một nhiệm vụ như sau:

    {my_task: 3, another_task: 1}

*my_task* sẽ có khả năng được thực thi 3 lần nhiều hơn so với *another_task*.

Nội bộ, từ điển trên sẽ thực sự được mở rộng thành một danh sách (và thuộc tính ``tasks`` được cập nhật) như sau:

    [my_task, my_task, my_task, another_task]

và sau đó :py:meth:`random.choice() <random.choice>` của Python được sử dụng để chọn nhiệm vụ từ danh sách.


.. _tagging-tasks:

@tag decorator
--------------

Bằng cách gắn thẻ các nhiệm vụ bằng cách sử dụng trang trí :py:func:`@tag <locust.tag>`, bạn có thể chọn lọc các nhiệm vụ được thực thi trong quá trình kiểm tra bằng cách sử dụng các đối số :code:`--tags` và :code:`--exclude-tags`. Xem xét ví dụ sau:

.. code-block:: python

    from locust import User, constant, task, tag

    class MyUser(User):
        wait_time = constant(1)

        @tag('tag1')
        @task
        def task1(self):
            pass

        @tag('tag1', 'tag2')
        @task
        def task2(self):
            pass

        @tag('tag3')
        @task
        def task3(self):
            pass

        @task
        def task4(self):
            pass

Nếu bạn bắt đầu bài kiểm tra này với :code:`--tags tag1`, chỉ *task1* và *task2* sẽ được thực thi trong quá trình kiểm tra. Nếu bạn bắt đầu nó với :code:`--tags tag2 tag3`, chỉ *task2* và *task3* sẽ được thực thi.

:code:`--exclude-tags` sẽ hoạt động theo cách ngược lại chính xác. Vì vậy, nếu bạn bắt đầu bài kiểm tra với :code:`--exclude-tags tag3`, chỉ *task1*, *task2*, và *task4* sẽ được thực thi. Sự loại trừ luôn thắng lợi so với việc bao gồm, vì vậy nếu một nhiệm vụ có một thẻ bạn đã bao gồm và một thẻ bạn đã loại trừ, nó sẽ không được thực thi.

Sự kiện
========

Nếu bạn muốn chạy một số mã thiết lập như một phần của bài kiểm tra của mình, thường đủ để đặt nó ở mức mô-đun của locustfile của bạn, nhưng đôi khi bạn cần làm một số việc vào thời gian cụ thể trong quá trình chạy. Để đáp ứng nhu cầu này, Locust cung cấp các sự kiện.

test_start và test_stop
-----------------------

Nếu bạn cần chạy một số mã khi bắt đầu hoặc kết thúc một bài kiểm tra tải, bạn nên sử dụng các sự kiện :py:attr:`test_start <locust.event.Events.test_start>` và :py:attr:`test_stop <locust.event.Events.test_stop>`. Bạn có thể thiết lập người nghe cho các sự kiện này ở mức mô-đun của locustfile của bạn:

.. code-block:: python

    from locust import events

    @events.test_start.add_listener
    def on_test_start(environment, **kwargs):
        print("A new test is starting")

    @events.test_stop.add_listener
    def on_test_stop(environment, **kwargs):
        print("A new test is ending")

init
----

Sự kiện ``init`` được kích hoạt ở đầu mỗi tiến trình Locust. Điều này đặc biệt hữu ích trong chế độ phân tán
nơi mỗi tiến trình worker (không phải mỗi người dùng) cần một cơ hội để thực hiện một số khởi tạo. Ví dụ, giả sử bạn có một số trạng thái toàn cục mà tất cả người dùng được tạo ra từ tiến trình này sẽ cần:

.. code-block:: python

    from locust import events
    from locust.runners import MasterRunner

    @events.init.add_listener
    def on_locust_init(environment, **kwargs):
        if isinstance(environment.runner, MasterRunner):
            print("I'm on master node")
        else:
            print("I'm on a worker or standalone node")

Các sự kiện khác
------------

Xem :ref:`mở rộng locust bằng cách sử dụng sự kiện <extending_locust>` để biết các sự kiện khác và ví dụ cụ thể về cách sử dụng chúng.

Lớp HttpUser
==============

:py:class:`HttpUser <locust.HttpUser>` là :py:class:`User <locust.User>` phổ biến nhất. Nó thêm một thuộc tính :py:attr:`client <locust.HttpUser.client>` được sử dụng để thực hiện các yêu cầu HTTP.

.. code-block:: python

    from locust import HttpUser, task, between

    class MyUser(HttpUser):
        wait_time = between(5, 15)

        @task(4)
        def index(self):
            self.client.get("/")

        @task(1)
        def about(self):
            self.client.get("/about/")


Thuộc tính client / HttpSession
------------------------------

:py:attr:`client <locust.HttpUser.client>` là một phiên bản của :py:class:`HttpSession <locust.clients.HttpSession>`. HttpSession là một lớp con/bọc cho
:py:class:`requests.Session`, vì vậy các tính năng của nó được tài liệu rõ ràng và nên quen thuộc với nhiều người. Những HttpSession thêm vào chủ yếu là báo cáo kết quả yêu cầu vào Locust (thành công/thất bại, thời gian phản hồi, độ dài phản hồi, tên).

Nó chứa các phương thức cho tất cả các phương thức HTTP: :py:meth:`get <locust.clients.HttpSession.get>`,
:py:meth:`post <locust.clients.HttpSession.post>`, :py:meth:`put <locust.clients.HttpSession.put>`,
...


Giống như :py:class:`requests.Session`, nó giữ nguyên cookie giữa các yêu cầu để dễ dàng đăng nhập vào các trang web.

.. code-block:: python
    :caption: Thực hiện một yêu cầu POST, xem phản hồi và sử dụng ngầm bất kỳ cookie phiên nào

    response = self.client.post("/login", {"username":"testuser", "password":"secret"})
    print("Response status code:", response.status_code)
    print("Response text:", response.text)
    response = self.client.get("/my-profile")

HttpSession bắt bất kỳ :py:class:`requests.RequestException <requests.RequestException>` nào được ném bởi Session (do lỗi kết nối, thời gian chờ hoặc tương tự), thay vào đó trả về một đối tượng phản hồi giả với *status_code* được đặt thành 0 và *content* được đặt thành None.

.. _catch-response:

Xác thực phản hồi (Validating responses)
--------------------

Yêu cầu được xem xét thành công nếu mã phản hồi HTTP là OK (<400), nhưng thường hữu ích để thực hiện một số xác thực bổ sung của phản hồi.

Bạn có thể đánh dấu một yêu cầu là thất bại bằng cách sử dụng đối số *catch_response*, một *with*-statement và một cuộc gọi đến *response.failure()*

.. code-block:: python

    with self.client.get("/", catch_response=True) as response:
        if response.text != "Success":
            response.failure("Got wrong response")
        elif response.elapsed.total_seconds() > 0.5:
            response.failure("Request took too long")


Bạn cũng có thể đánh dấu một yêu cầu là thành công, ngay cả khi mã phản hồi không tốt:

.. code-block:: python

    with self.client.get("/does_not_exist/", catch_response=True) as response:
        if response.status_code == 404:
            response.success()

Bạn cũng có thể tránh việc đăng ký một yêu cầu bằng cách ném một ngoại lệ và sau đó bắt nó bên ngoài khối với. Hoặc bạn có thể ném một :ref:`ngoại lệ locust <exceptions>`, như trong ví dụ dưới đây, và để Locust bắt nó.

.. code-block:: python

    from locust.exception import RescheduleTask
    ...
    with self.client.get("/does_not_exist/", catch_response=True) as response:
        if response.status_code == 404:
            raise RescheduleTask()

REST/JSON APIs
--------------

Nếu bạn làm việc với REST hoặc JSON APIs, bạn có thể muốn kiểm tra phản hồi JSON. :ref:`FastHttpUser <rest>` cung cấp một phương thức sẵn có ``rest``, nhưng bạn cũng có thể tự làm:

.. code-block:: python

    from json import JSONDecodeError
    ...
    with self.client.post("/", json={"foo": 42, "bar": None}, catch_response=True) as response:
        try:
            if response.json()["greeting"] != "hello":
                response.failure("Did not get expected value in greeting")
        except JSONDecodeError:
            response.failure("Response could not be decoded as JSON")
        except KeyError:
            response.failure("Response did not contain expected key 'greeting'")

.. _name-parameter:

Nhóm các yêu cầu
-----------------

Rất phổ biến đối với các trang web để có các trang có URL chứa một số tham số động.
Thường thì việc nhóm các URL này lại với nhau trong thống kê người dùng. Điều này có thể được thực hiện
bằng cách truyền một đối số *name* vào các phương thức yêu cầu của :py:class:`HttpSession <locust.clients.HttpSession>`.

Ví dụ:

.. code-block:: python

    # Thống kê cho các yêu cầu này sẽ được nhóm dưới: /blog/?id=[id]
    for i in range(10):
        self.client.get("/blog?id=%i" % i, name="/blog?id=[id]")

Có thể có tình huống mà việc truyền một tham số vào hàm yêu cầu không thể thực hiện được, chẳng hạn khi tương tác với thư viện/SDK bọc một phiên Requests.
Một cách thay thế để nhóm các yêu cầu là sử dụng thuộc tính ``client.request_name``.

.. code-block:: python

    # Thống kê cho các yêu cầu này sẽ được nhóm dưới: /blog/?id=[id]
    self.client.request_name="/blog?id=[id]"
    for i in range(10):
        self.client.get("/blog?id=%i" % i)
    self.client.request_name=None

Nếu bạn muốn liên kết nhiều nhóm với ít mã tối thiểu, bạn có thể sử dụng trình quản lý ngữ cảnh ``client.rename_request()``.

.. code-block:: python

    @task
    def multiple_groupings_example(self):
        # Thống kê cho các yêu cầu này sẽ được nhóm dưới: /blog/?id=[id]
        with self.client.rename_request("/blog?id=[id]"):
            for i in range(10):
                self.client.get("/blog?id=%i" % i)

        # Thống kê cho các yêu cầu này sẽ được nhóm dưới: /article/?id=[id]
        with self.client.rename_request("/article?id=[id]"):
            for i in range(10):
                self.client.get("/article?id=%i" % i)

Sử dụng :ref:`catch_response <catch-response>` và truy cập `request_meta <https://github.com/locustio/locust/blob/master/locust/clients.py#L145>`_ trực tiếp, bạn cũng có thể đổi tên yêu cầu dựa trên một số thông tin trong phản hồi.

.. code-block:: python

    with self.client.get("/", catch_response=True) as resp:
        resp.request_meta["name"] = resp.json()["name"]


Cài đặt Proxy HTTP
-------------------
Để cải thiện hiệu suất, chúng tôi cấu hình yêu cầu để không tìm kiếm cài đặt proxy HTTP trong môi trường bằng cách đặt
thuộc tính trust_env của requests.Session thành ``False``. Nếu bạn không muốn điều này, bạn có thể đặt
``locust_instance.client.trust_env`` thành ``True``. Để biết thêm chi tiết, hãy tham khảo
`tài liệu của requests <https://requests.readthedocs.io/en/master/api/#requests.Session.trust_env>`_.

Connection reuse
----------------

Mặc định, các kết nối được tái sử dụng bởi một HttpUser, ngay cả qua các chạy nhiệm vụ. Để tránh việc tái sử dụng kết nối, bạn có thể làm như sau:

.. code-block:: python

    self.client.get("/", headers={"Connection": "close"})
    self.client.get("/new_connection_here")

Hoặc bạn có thể đóng toàn bộ đối tượng requests.Session (điều này cũng xóa cookie, đóng phiên SSL vv). Điều này có một số chi phí CPU
(và thời gian phản hồi của yêu cầu tiếp theo sẽ cao hơn do việc tái thỏa thuận SSL vv), vì vậy không sử dụng điều này trừ khi bạn thực sự cần.

.. code-block:: python

    self.client.get("/")
    self.client.close()
    self.client.get("/new_connection_here")


Connection pooling
------------------

Vì mỗi :py:class:`HttpUser <locust.HttpUser>` tạo ra một :py:class:`HttpSession <locust.clients.HttpSession>`,
mỗi thể hiện người dùng có một pool kết nối riêng. Điều này tương tự như cách người dùng thực sự (trình duyệt) tương tác với máy chủ web.

Nếu thay vào đó bạn muốn chia sẻ kết nối, bạn có thể sử dụng một quản lý pool duy nhất. Để làm điều này, đặt :py:attr:`pool_manager <locust.HttpUser.pool_manager>`
là một phiên bản của :py:class:`urllib3.PoolManager`.

.. code-block:: python

    from locust import HttpUser
    from urllib3 import PoolManager

    class MyUser(HttpUser):
        # Tất cả các thể hiện của lớp này sẽ bị giới hạn tối đa 10 kết nối đồng thời.
        pool_manager = PoolManager(maxsize=10, block=True)

Để biết thêm tùy chọn cấu hình, hãy tham khảo
`tài liệu của urllib3 <https://urllib3.readthedocs.io/en/stable/reference/urllib3.poolmanager.html>`_.

TaskSets
================================
TaskSets là một cách để cấu trúc các bài kiểm tra của các trang web/hệ thống có cấu trúc phân cấp. Bạn có thể :ref:`đọc thêm về nó ở đây <tasksets>`.

Ví dụ
========

Có rất nhiều ví dụ về locustfile `ở đây <https://github.com/locustio/locust/tree/master/examples>`_.

Cách cấu trúc mã kiểm tra của bạn
================================

Quan trọng nhớ rằng locustfile.py chỉ là một mô-đun Python thông thường được nhập bởi Locust. Từ mô-đun này, bạn có thể nhập mã Python khác như bạn thường làm trong bất kỳ chương trình Python nào. Thư mục làm việc hiện tại được tự động thêm vào ``sys.path`` của python, vì vậy bất kỳ tệp/thư mục mô-đun/packges nào nằm trong thư mục làm việc có thể được nhập bằng câu lệnh ``import`` của python.

Đối với các bài kiểm tra nhỏ, giữ tất cả mã kiểm tra trong một ``locustfile.py`` duy nhất có thể hoạt động tốt, nhưng đối với các bài kiểm tra lớn hơn, bạn có thể muốn chia mã thành nhiều tệp và thư mục.

Cách bạn cấu trúc mã nguồn kiểm tra là hoàn toàn tùy thuộc vào bạn, nhưng chúng tôi khuyến nghị bạn nên tuân thủ các quy tắc tốt nhất của Python. Dưới đây là một cấu trúc tệp mẫu của một dự án Locust ảo:

* Thư mục gốc

  * ``common/``

    * ``__init__.py``
    * ``auth.py``
    * ``config.py``
  * ``locustfile.py``
  * ``requirements.txt`` (Các phụ thuộc Python bên ngoài thường được giữ trong một requirements.txt)

Một dự án với nhiều locustfiles cũng có thể giữ chúng trong một thư mục con:

* Thư mục gốc

  * ``common/``

    * ``__init__.py``
    * ``auth.py``
    * ``config.py``
  * ``my_locustfiles/``

    * ``api.py``
    * ``website.py``
  * ``requirements.txt``


Với bất kỳ cấu trúc dự án nào ở trên, locustfile của bạn có thể import các thư viện chung bằng cách sử dụng:

.. code-block:: python

    import common.auth
