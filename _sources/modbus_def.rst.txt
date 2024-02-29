Defination of Modbus
====================

Defination
~~~~~~~~~~

Nói một cách đơn giản, nó là một phương pháp được sử dụng để truyền thông tin qua đường nối tiếp giữa các thiết bị điện tử. Thiết bị yêu cầu thông tin được gọi là Modbus Client và các thiết bị cung cấp thông tin được gọi là Modbus Server. Trong mạng Modbus tiêu chuẩn, có một Modbus Client và tối đa 247 Modbus Servers, mỗi Modbus Server có một Địa chỉ Modbus Server duy nhất từ 1 đến 247. Modbus Client cũng có thể ghi thông tin vào Modbus Server.

Chúng ta thường dùng Modbus ở đâu?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Modbus là một protocol mở, có nghĩa là nó miễn phí cho những nhà sản xuất có thể xây dựng trên thiết bị của họ mà không cần trả phí. Nó đã trở thành một giao thức truyền thông tiêu chuẩn trong công nghiệp và hiện là phương tiện phổ biến nhất để kết nối các thiết bị điện tử công nghiệp. Nó được sử dụng rộng rãi bởi nhiều nhà sản xuất trong nhiều ngành công nghiệp. Modbus thường được sử dụng để truyền tín hiệu từ thiết bị đo đạc và điều khiển trở lại bộ điều khiển chính hoặc hệ thống thu thập dữ liệu, ví dụ như hệ thống đo nhiệt độ và độ ẩm và truyền kết quả đến máy tính.

Modbus thường được sử dụng để kết nối máy tính giám sát với thiết bị đầu cuối từ xa (Remote Terminal Unit - RTU), trong các hệ thống SCADA. 


Modbus hoạt động như thế nào?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Modbus được truyền qua đường nối tiếp giữa các thiết bị. Thiết lập đơn giản nhất sẽ là một cáp nối tiếp duy nhất kết nối các cổng nối tiếp trên hai thiết bị, Client và Server.

.. figure:: imagines/modbus_1.png
   :align: center
   :alt: Priority
   :scale: 100%

Dữ liệu được gửi dưới dạng chuỗi số 1 và số 0 được gọi là bit. Mỗi bit được gửi dưới dạng điện áp. Các số 0 được gửi dưới dạng điện áp dương và số 1 là âm. Các bit được gửi rất nhanh. Tốc độ truyền thông thường là 9600 baud (bit trên giây).


Data được lưu trong Modbus (tiêu chuẩn) như thế nào?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Thông tin được lưu trữ trong Server ở bốn bảng khác nhau. Hai bảng lưu trữ các giá trị bật/tắt riêng biệt (coils) và hai bảng lưu trữ các giá trị số (thanh ghi). Mỗi cuộn dây và thanh ghi đều có một bảng chỉ đọc và bảng đọc-ghi. Mỗi bảng có 9999 giá trị. Mỗi cuộn dây hoặc tiếp điểm là 1 bit và được gán một địa chỉ dữ liệu trong khoảng từ 0000 đến 270E.  Mỗi thanh ghi là 1 word = 16 bit = 2 byte và cũng có địa chỉ dữ liệu trong khoảng từ 0000 đến 270E.

.. figure:: imagines/modbus_2.png
   :align: center
   :alt: Priority
   :scale: 100%

Coil/Register Numbers ó thể được coi là tên vị trí vì chúng không xuất hiện trong tin nhắn thực tế. Data Addresses được sử dụng trong tin nhắn. Ví dụ: Thanh ghi lưu trữ đầu tiên, số 40001, có Địa chỉ dữ liệu 0000. Sự khác biệt giữa hai giá trị này là phần bù. Mỗi bảng có một độ lệch khác nhau. 1, 10001, 30001 và 40001.

Server ID là gì?
~~~~~~~~~~~~~~~~

Mỗi server trong mạng được gán một địa chỉ đơn vị duy nhất từ 1 đến 247. Khi Client yêu cầu dữ liệu, byte đầu tiên nó gửi là địa chỉ Server. Bằng cách này, mỗi Server sẽ biết sau byte đầu tiên có bỏ qua tin nhắn hay không.


