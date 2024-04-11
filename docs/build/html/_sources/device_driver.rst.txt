Device driver model
===================

Zephyr RTOS dựa trên nRF Connect SDK cung cấp mô hình driver thiết bị (**device driver model**).
Việc triển khai trình điều khiển này được tách biệt hoàn toàn khỏi API của nó, cho phép các nhà phát triển tắt việc triển khai trình điều khiển đòn bẩy thấp mà không cần sửa đổi ứng dụng ở trên cùng, vì chúng tôi có thể sử dụng cùng một API chung.

Việc tách rời này có nhiều lợi ích, bao gồm mức độ di động cao, vì nó có thể sử dụng cùng một mã trên các bo mạch khác nhau mà không cần sửa đổi cách triển khai trình điều khiển cơ bản theo cách thủ công.

Zephyr device driver model là kiến trúc xác định mối liên kết giữa các API chung và việc triển khai device driver. Chúng ta có thể chia mô hình thành ba phần khác nhau, như trong hình bên dưới.

.. figure:: imagines/device_driver_01.png
   :align: center
   :alt: Priority
   :scale: 100%

Ba sub-categories cho device driver model là:

1. Device driver APIs
2. Device driver instances
3. Device driver implementation

Ví dụ về Device driver model
----------------------------

Để tương tác với thiết bị ngoại vi phần cứng hoặc system block, chúng ta cần sử dụng device driver, đây là phần mềm xử lý các chi tiết cấp thấp trong việc định cấu hình phần cứng theo cách chúng ta muốn.
Về cơ bản, chúng ta có thể tắt việc triển khai low-level driver mà không cần sửa đổi ứng dụng vì chúng ta có thể sử dụng cùng một API chung.

Việc tách rời này có nhiều lợi ích, bao gồm mức độ di động cao, vì nó có thể sử dụng cùng một mã trên các bo mạch khác nhau mà không cần phải sửa đổi cách triển khai trình điều khiển cơ bản theo cách thủ công.

Ứng dụng tương tác với phần cứng thông qua API chung bằng cách lấy con trỏ thiết bị cho phần cứng được đề cập, sử dụng macro **DEVICE_DT_GET()** hoặc các macro liên quan.

.. admonition:: Ghi chú
   :class: note

   Cũng có thể lấy con trỏ thiết bị thông qua device_get_bind(), mặc dù điều này không còn được khuyến khích nữa.

   Ngược lại với cách sử dụng trước đây là sử dụng device_get_bind() để truy xuất con trỏ thiết bị, việc sử dụng DEVICE_DT_GET() có lợi là sẽ thất bại tại thời điểm build nếu thiết bị không được driver cấp phát, chẳng hạn như nếu thiết bị không tồn tại trong devicetree hoặc có trạng thái bị vô hiệu hóa.

   Ngoài ra, không giống như device_get_bind(), nó không thực hiện so sánh run-time string, điều này có thể ảnh hưởng đến hiệu suất trong một số trường hợp.

Zephyr device model chịu trách nhiệm liên kết giữa các API chung và triển khai device driver.

.. figure:: imagines/device_driver_02.png
   :align: center
   :alt: Priority
   :scale: 100%

.. figure:: imagines/device_driver_03.png
   :align: center
   :alt: Priority
   :scale: 100%

Để lấy con trỏ thiết bị, bạn cần chuyển mã định danh devicetree node. Như đã đề cập trong phần Devicetree, có nhiều cách để lấy mã định danh node. Hai cách phổ biến là gắn nhãn node thông qua macro DT_NODELABEL() và bằng bí danh thông qua macro DT_ALIAS().

Trước khi sử dụng con trỏ thiết bị, cần kiểm tra nó bằng device_is_ready():

.. figure:: imagines/device_driver_04.png
   :align: center
   :alt: Priority
   :scale: 100%

**device_is_ready()** sẽ kiểm tra xem thiết bị đã sẵn sàng để sử dụng hay chưa, ví dụ: thiết bị có được khởi tạo đúng cách hay không.

Đoạn code sau sẽ lấy mã định danh devicetree node được trả về bởi DT_NODELABEL() và trả về một con trỏ tới device object. Sau đó, device_is_ready() xác minh rằng thiết bị đã sẵn sàng để sử dụng, tức là ở trạng thái có thể sử dụng được với API tiêu chuẩn của nó.

.. code-block:: c
   
   const struct device *dev;
   dev = DEVICE_DT_GET(DT_NODELABEL(uart0));
   if (!device_is_ready(dev)) {
      return;
   }

Để sử dụng API chung của trình điều khiển thiết bị, bạn phải có một con trỏ kiểu const struct device để trỏ đến việc triển khai nó. Bạn cần thực hiện việc này cho mỗi ngoại vi.
Ví dụ: nếu bạn có hai thiết bị ngoại vi UART (&uart0 và &uart1) và bạn muốn sử dụng cả hai, bạn phải có hai con trỏ riêng biệt thuộc loại const struct device.

Nói cách khác, bạn cần có hai lệnh gọi khác nhau tới DEVICE_DT_GET().

.. admonition:: Ghi chú
   :class: note

   Hầu hết các API ngoại vi sẽ có các API tương đương với DEVICE_DT_GET() và device_is_ready() dành riêng cho thiết bị ngoại vi. Ví dụ: GPIO, GPIO_DT_SPEC_GET() và gpio_is_ready_dt() . Đây là những cách được khuyến nghị để sử dụng thiết bị ngoại vi vì chúng thu thập thêm thông tin về thiết bị ngoại vi từ cấu trúc cây thiết bị và giảm nhu cầu thêm cấu hình ngoại vi trong mã ứng dụng.

Thực thi device driver
----------------------

Khi triển khai device driver, người ta cần xem xét một số thành phần chính. Hầu hết các chi tiết triển khai của device driver được bao quanh xung quanh cấu trúc dữ liệu trình điều khiển, thiết bị cấu trúc, được hiển thị bên dưới trong đoạn mã đơn giản hóa.

.. code-block:: c

   struct device {
      const char *name;
      const void *config;
      const void *api;
      void * const data;
   };

DEVICE_DEFINE() là macro được sử dụng để tạo một struct device, chứa tất cả thông tin và cách tương tác với thiết bị cơ bản. Một số thành phần chính được đề cập dưới đây được cắm vào hệ thống bằng macro này.

Các thành phần chính của Zephyr Device Driver
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Device tree điển hình trong Zephyr bao gồm một số thành phần chính.

Cấu hình: Một trong những thành phần của cấu trúc dữ liệu driver, **const void * config**, lưu trữ dữ liệu cấu hình của thiết bị và ở chế độ chỉ đọc. Dữ liệu cấu hình này có thể khác nhau đối với các thiết bị khác nhau và chứa đầy thông tin khi triển khai driver thiết bị tại thời điểm khởi tạo driver.

Khởi tạo: Trong quá trình khởi tạo, driver thiết lập các tài nguyên cần thiết (như nguồn, bảo mật và trạng thái ban đầu của chân và bộ nhớ). Quá trình khởi tạo driver thiết bị phải chạy trước khi luồng chính của ứng dụng chạy và Zephyr cung cấp **SYS_INIT()** để thực hiện các lệnh gọi khởi tạo như vậy tại thời điểm khởi động.

APIs: Driver hiển thị một bộ API cho phép mã ứng dụng tương tác với phần cứng. API này bên trong driver thiết bị được tạo dưới dạng cấu trúc của các con trỏ hàm. Địa chỉ của cấu trúc này được chuyển đến tham số api của cấu trúc dữ liệu driver trong macro **DEVICE_DEFINE()** 
tại thời điểm định nghĩa thiết bị trong quá trình triển khai driver thiết bị. Các API này bao gồm các chức năng đọc dữ liệu cảm biến, điều khiển bộ truyền động, định cấu hình các thanh ghi, v.v.

Interrupt Handlers: Đối với các thiết bị tạo ra ngắt (ví dụ: chân GPIO, thiết bị ngoại vi, bộ hẹn giờ), interrupt handlers được triển khai bên trong driver thiết bị để xử lý các sự kiện này một cách không đồng bộ và thông báo cho ứng dụng về những thay đổi xảy ra thông qua API driver.

Power Management: Một số driver hỗ trợ tính năng quản lý nguồn để tối ưu hóa mức tiêu thụ năng lượng khi xử lý các thiết bị chạy bằng pin. Nếu driver thiết bị đã triển khai cần hoặc muốn hỗ trợ trình cắm vào quản lý nguồn điện của hệ thống Zephyr thì driver thiết bị cần triển khai 
và hiển thị việc triển khai **struct pm_device** và chuyển nó tới macro **DEVICE_DEFINE()** tại thời điểm định nghĩa thiết bị . NULL cần được chuyển tới DEVICE_DEFINE nếu driver không muốn triển khai quản lý nguồn cho thiết bị.

Viết device driver bị trong Zephyr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Để viết device driver tùy chỉnh trong Zephyr:

1. Bắt đầu bằng cách tạo một thư mục mới trong trình điều khiển trong cây nguồn của dự án của bạn.

2. Triển khai các chức năng cần thiết như quy trình khởi tạo và API dành riêng cho nền tảng phần cứng mục tiêu của bạn.

3. Add your driver to the Zephyr build system by modifying the configuration file (prj.conf or zephyr.dts) and specifying the driver in the device tree.

4. Sử dụng các API thích hợp từ các hệ thống con Zephyr hiện có, chẳng hạn như GPIO, I2C, SPI, v.v., để tương tác với các thiết bị phần cứng.

Zephyr cung cấp một bộ trình điều khiển phong phú cho các thiết bị ngoại vi và giao diện truyền thông khác nhau. Các driver dựng sẵn này có thể được sử dụng trực tiếp hoặc dùng làm tài liệu tham khảo khi xây dựng driver tùy chỉnh.

System initialization
~~~~~~~~~~~~~~~~~~~~~

Trong Zephyr, bạn có thể khởi tạo các mô-đun của mình (thư viện/trình điều khiển, v.v.) theo thứ tự nhất định để giải quyết các phụ thuộc lẫn nhau. Ví dụ: giả sử việc khởi tạo driver X phụ thuộc vào driver Y được khởi tạo trước. 
Trong trường hợp đó, chúng ta cần chọn thứ tự khởi tạo driver Y sẽ được thực hiện trước khi quá trình khởi tạo driver X bắt đầu. Việc khởi tạo các mô-đun có thể được thực hiện bằng cách sử dụng SYS_INIT() lấy mức độ khởi tạo và mức độ ưu tiên làm đối số để tuần tự hóa thứ tự khởi tạo. 
Các mức khởi tạo được mô tả rõ ràng trong tài liệu Zephyr.


System call
~~~~~~~~~~~

Trong Zephyr, giống như bất kỳ hệ điều hành nào khác, system là cách để các chương trình tương tác với hệ điều hành. 
Nó cho phép họ yêu cầu các dịch vụ như thao tác tệp, quản lý quy trình và giao tiếp mạng không có sẵn trực tiếp trong mã của riêng họ.

Các system call cung cấp cầu nối giữa các ứng dụng cấp người dùng và nhân của hệ điều hành. Chúng cho phép các chương trình truy cập các
hướng dẫn và tài nguyên đặc quyền mà lẽ ra không thể truy cập được từ không gian người dùng.

Trong hầu hết các ngôn ngữ lập trình, các system call được thực hiện thông qua giao diện lập trình ứng dụng (API). API này cung cấp một tập 
hợp các hàm hoặc phương thức mà nhà phát triển có thể gọi để sử dụng các khả năng cơ bản của hệ thống.

Một số ví dụ phổ biến về system call bao gồm:

* Mở và đóng các thiết bị ngoại vi/tập tin

* Đọc và ghi vào thiết bị ngoại vi/tập tin

* Tạo và quản lý các quy trình

* Phân bổ bộ nhớ động

* Gửi dữ liệu qua mọi phương tiện truyền tải (không dây/ổ cắm/I2C/SPI, v.v.)

System call thường liên quan đến việc truyền tham số hoặc đối số để chỉ định hành vi mong muốn. Các tham số này có thể bao gồm tên tệp, kích thước bộ đệm, ID tiến trình hoặc thông tin liên quan khác.

User mode
~~~~~~~~~

Trong chế độ người dùng của Zephyr, các ứng dụng chạy với các đặc quyền hạn chế để đảm bảo tính ổn định và bảo mật của hệ thống. Ở chế độ này, các ứng dụng có quyền truy cập hạn chế vào tài nguyên phần cứng và bị cô lập khỏi các tiến trình khác đang chạy trong hệ thống. 
Điều này giúp ngăn chặn mã độc hoặc mã lỗi ảnh hưởng đến các hoạt động quan trọng của hệ thống.

Supervisor mode
~~~~~~~~~~~~~~~

Supervisor mode, còn được gọi là Kernel mode, là môi trường thực thi đặc quyền chứa kernel hệ điều hành. Trong chế độ này, kernel có quyền truy cập không hạn chế vào tất cả các tài nguyên phần cứng và có thể thực hiện các hoạt động đặc quyền như quản lý bộ nhớ,
 lập lịch tác vụ, xử lý các ngắt và điều khiển trình điều khiển thiết bị. Chỉ mã đáng tin cậy mới được thực thi ở chế độ giám sát để duy trì tính toàn vẹn của hệ thống.

Cả ứng dụng ở user mode và supervisor mode đều cùng tồn tại trong hệ điều hành Zephyr để cung cấp nền tảng mạnh mẽ để phát triển các hệ thống nhúng với các yêu cầu nghiêm ngặt về hiệu suất, độ tin cậy và bảo mật.

Thông thường, khi một chương trình thực hiện lệnh gọi hệ thống, nó sẽ chuyển từ chế độ người dùng sang chế độ giám sát, trong đó thao tác được yêu cầu được thực thi thay mặt cho chương trình gọi. Sau khi hoàn thành, quyền kiểm soát sẽ được trả lại cho người gọi cùng với mọi kết quả hoặc mã lỗi. 
Sự khác biệt giữa chế độ người dùng và chế độ giám sát thường được thực hiện dựa trên một số mục tiêu thiết kế bảo mật.

Device driver invocation context
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hiểu ngữ cảnh gọi của các cuộc gọi hệ thống là điều cần thiết đối với các nhà phát triển muốn xây dựng các thành phần phần mềm cấp thấp hoặc tương tác trực tiếp với các thiết bị phần cứng. Hầu hết các mẫu được tạo trong ứng dụng hoạt động với CONFIG_USERSPACE chưa được bật. 
Ở chế độ này, tất cả các API chỉ gọi trực tiếp chức năng triển khai.

Driver Data Structures
----------------------

Các macro khởi tạo thiết bị đưa vào một số data structures tại thời điểm build, được chia thành các phần chỉ đọc và phần có thể thay đổi trong thời gian chạy.

.. code-block:: c

   struct device {
      const char *name;
      const void *config;
      const void *api;
      void * const data;
   };

Những thành phần ``config`` thì cho những data được cấu hình read-only. Ví dụ: các địa chỉ IO, IRQ line number, ... Cấu hình này được ``config pointer`` truyền tới ``DEVICE_DEFINE()`` và các macro có liên quan.

Những struct ``data`` thì được đặt trong RAM, và được sử dụng bởi driver để quản lý runtime. Ví dụ: nó có thể chứa số lượng tham chiếu, semaphores, scratch buffers,...

NHững struct ``api`` ánh xạ những API từ subsystem tới các driver thực thi cho từng thiết bị cụ thể. 

Subsystems and API Structures
-----------------------------

Hầu hết các driver sẽ thực hiện subsystem API độc lập với thiết bị.

Một subsystem API thường được định nghĩa như thế này:

.. code-block:: c

   typedef int (*subsystem_do_this_t)(const struct device *dev, int foo, int bar);
   typedef void (*subsystem_do_that_t)(const struct device *dev, void *baz);

   struct subsystem_api {
         subsystem_do_this_t do_this;
         subsystem_do_that_t do_that;
   };

   static inline int subsystem_do_this(const struct device *dev, int foo, int bar)
   {
         struct subsystem_api *api;

         api = (struct subsystem_api *)dev->api;
         return api->do_this(dev, foo, bar);
   }

   static inline void subsystem_do_that(const struct device *dev, void *baz)
   {
         struct subsystem_api *api;

         api = (struct subsystem_api *)dev->api;
         api->do_that(dev, baz);
   }

Ví dụ cụ thể:

.. code-block:: c
   
   static int my_driver_do_this(const struct device *dev, int foo, int bar)
   {
         ...
   }

   static void my_driver_do_that(const struct device *dev, void *baz)
   {
         ...
   }

   static struct subsystem_api my_driver_api_funcs = {
         .do_this = my_driver_do_this,
         .do_that = my_driver_do_that
   };

Lúc này driver sẽ chuyển ``my_driver_api_funcs`` làm ``api`` trong ``DEVICE_DEFINE()``.



