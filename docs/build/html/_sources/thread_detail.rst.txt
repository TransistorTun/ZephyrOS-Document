Detail of Thread
================

Phần này miêu tả và hướng dẫn các service kernel để  tạo, lập lịch và xóa các thread thực thi độc lập.

Một thread là kernel object được sử dụng để xử lý ứng dụng quá dài hoặc quá phức tạp được thực hiện bởi ISR.


Số lượng những thread có thể xác định bởi một ứng dụng (thường thì chỉ bị giới hạn bởi RAM). 

Mỗi thread được tham chiếu bởi một thread id, thread id được chỉ định khi thread được tạo.

Một thread có những thuộc tính quan trọng sau:

    * STACK AREA: là một vùng của memory được dùng để làm stack cho thread. Kích cỡ của stack area có thể được điều chỉnh để phù hợp với nhu cầu thực tế của quá trình xử lý thread. Các macro đặc biệt tồn tại để tạo và làm việc với các vùng stack memory
    * THREAD CONTROL BLOCK: 
    * ENTRY POINT FUNCTION:
    * SCHEDULING PRIORITY: 
    * 
    * START DELAY:
    * EXECUTION MODE: 

Life cycle
----------

Thread Creation
~~~~~~~~~~~~~~~

Thread phải được tạo trước khi có thể sử dụng. Kernel khởi tạo "thread control block" ở phần đầu của stack. Phần stack còn lại của thread thường đươc để thành uninitialize.

**k_no_wait** sẽ hướng dẫn kernel tạo thread ngay lập tức. Ngoài ra kernel có thể được chỉ định thời gian tạo thread ví dụ cho như là chỉ định thời gian cho nó, để có thời gian để phần cứng thiết bị trở nên khả dụng.

Kernel cho phép cái delayed start có thể được hủy trước khi thread được thực hiện. Việc yêu cầu hủy thread có không có hiệu lực khi mà thread đã bắt đầu. Khi đã hủy một cái thread thì muốn sử dụng lại cần khởi tạo lại từ đầu.

Thread Termination
~~~~~~~~~~~~~~~~~~

Một khi thread được bắt đầu, nó thường được thực thi vĩnh viễn. Tuy nhiên, một thread có thể kết thúc việc thực thi đồng bộ của nó bằng cách quay về hàm entry point của nó. Việc này gọi là **termination** (chấm dứt).

Một thread khi kết thúc có trách nhiệm giải phóng mọi tài nguyên mà được chia sẻ mà nó có thể sở hữu (chẳng hạn như mutex và các bộ nhớ cấp phát động của nó) trước khi quay trở lại, vì kernel không thể tự lấy lại những tài nguyên đó.

Trong một số trường hợp, thread muốn sleep cho tới khi một thread khác dừng hẳn. Điều này có thể thực hiện bởi API **k_thread_join()**. Nó sẽ ngừng goị thread cho đến khi hết thời gian chờ. Thread mục tiêu tự thoát, hoặc bị hủy bỏ (do lệnh gọi k_thread_abort() hoặc gây ra lỗi nghiêm trọng).

Khi một thread đã kết thúc, kernel đảm bảo rằng cấu trúc thread sẽ không được sử dụng. Bộ nhớ của cấu trúc như vậy sau đó có thể được sử dụng lại cho bất kỳ mục đích nào, bao gồm cả việc tạo ra một thread mới. Lưu ý rằng thread phải được kết thúc hoàn toàn, điều này thể hiện các điều kiện chạy đua trong đó việc hoàn thành tín hiệu logic của chính thread đó được một thread khác nhìn thấy trước khi quá trình xử lý kernel hoàn tất. Trong trường hợp thông thường, code nên sử dụng **k_thread_join()** hoặc **k_thread_abort()** để đồng bộ hóa ở trạng thái kết thúc thread và không dựa vào tín hiệu từ bên trong logic ứng dụng.

Thread Aborting
~~~~~~~~~~~~~~~

Một thread có thể kết thúc quá trình thực thi của nó một cách không đồng bộ bằng cách hủy bỏ (abort). Kernel tự động hủy bỏ một thread nếu thread đó kích hoạt một tình trạng lỗi nghiêm trọng, chẳng hạn như hủy tham chiếu một con trỏ null.

Một thread cũng có thể bị hủy bỏ bởi một thread khác (hoặc bởi chính nó) bằng cách gọi k_thread_abort(). Tuy nhiên, thông thường tốt hơn là báo hiệu một thread tự kết thúc một cách nhẹ nhàng hơn là hủy bỏ nó.

Giống như việc chấm dứt thread, kernel không thể tự lấy lại các tài nguyên được chia sẻ thuộc sở hữu của một thread bị hủy bỏ.

.. admonition:: Ghi chú
   :class: note

   Kernel hiện không đưa ra bất kỳ xác nhận nào liên quan đến khả năng của ứng dụng trong việc khôi phục một thread bị hủy bỏ.

Thread Suspension
~~~~~~~~~~~~~~~~~

Một thread có thể bị ngăn không cho thực thi trong một khoảng thời gian không xác định nếu nó bị suspend. Hàm **k_thread_suspend()** có thể được sử dụng để tạm dừng bất kỳ thread nào, bao gồm cả thread đang gọi. Việc suspend một thread đã bị suspend (tạm dừng) không có tác dụng gì.

Sau khi bị suspend, một thread không thể được lên lịch (schedule) cho đến khi một thread khác gọi **k_thread_resume()** để xóa việc tạm dừng.

.. admonition:: Ghi chú
   :class: note

   Một thread có thể ngăn chính nó thực thi trong một khoảng thời gian nhất định bằng cách sử dụng **k_sleep()**. Tuy nhiên, điều này khác với việc suspend một thread vì một thread đang ngủ sẽ tự động được thực thi khi đạt đến giới hạn thời gian.

Thread State
------------

Một thread không có yếu tố ngăn cản việc thực thi nó được coi là **sẵn sàng** và đủ điều kiện để được chọn làm thread hiện tại.

Một thread có một hoặc nhiều yếu tố ngăn cản việc thực thi nó được coi là **chưa sẵn sàng** và không thể được chọn làm thread hiện tại.

Các yếu tố làm một thread **unready** (chưa sẵn sàng):

    * Thread chưa được bắt đầu
    * Thread đang chờ kernel hoàn thành một thao tác nào đó. Ví dụ: Thread đang sử dụng một semaphore không có sẵn.
    * Thread đang chờ thời gian để xuất hiện
    * Thread đang bị suspend
    * Thread đang bị tạm dừng hoặc chấm dứt

.. figure:: imagines/thread_state.svg
   :align: center
   :alt: Priority
   :scale: 100%   

.. admonition:: Ghi chú
   :class: note

    Mặc dù sơ đồ trên có thể gợi ý rằng cả trạng thái Ready và Running đều là các trạng thái thread riêng biệt, nhưng đó không phải là cách giải thích chính xác. Ready là trạng thái của thread và Running là trạng thái của Schedule chỉ áp dụng cho các thread Ready.

Thread Stack object
-------------------

Mỗi thread yêu cầu stack buffer riêng để CPU đẩy context (ngữ cảnh). Tùy thuộc vào cấu hình, có một số ràng buộc phải được đáp ứng:

    * Có thể cần thêm bộ nhớ dành riêng cho cấu trúc quản lý bộ nhớ (memory management structure).
    * Nếu tính năng phát hiện stack overflow dựa trên bảo vệ được bật, một vùng quản lý bộ nhớ được bảo vệ ghi nhỏ phải ngay trước stack buffer để phát hiện tràn (stack overflow).
    * Nếu userspace được bật, một stack nâng cao đặc quyền có kích thước cố định riêng biệt phải được dành riêng để phục vụ như một kernel stack riêng để xử lý các system calls.
    * Nếu userspace được bật, stack buffer của thread phải có kích thước và căn chỉnh phù hợp sao cho vùng bảo vệ bộ nhớ có thể được lập trình phù hợp chính xác.

Các ràng buộc căn chỉnh có thể khá hạn chế, ví dụ: một số MPU yêu cầu các vùng của chúng phải có lũy thừa bằng hai về kích thước và được căn chỉnh theo kích thước của chính nó.

Kernel-only Stacks
~~~~~~~~~~~~~~~~~~

Thread stacks
~~~~~~~~~~~~~

Thread Priorities
-----------------

Mức độ ưu tiên của thread là một giá trị nguyên và có thể âm hoặc không âm. Mức độ ưu tiên thấp hơn về số lượng sẽ được ưu tiên hơn các giá trị cao hơn về số lượng. Ví dụ: scheduler cung cấp cho thread A có mức độ ưu tiên 4 cao hơn thread B có mức độ ưu tiên 7; tương tự như vậy, thread C có mức độ ưu tiên -2 có mức độ ưu tiên cao hơn cả thread A và thread B.

Scheduler phân biệt giữa hai loại thread, dựa trên mức độ ưu tiên của từng thread.

    * Một *cooperative* thread có giá trị ưu tiên âm. Khi nó trở thành thread hiện tại, một cooperative thread vẫn là thread hiện tại cho đến khi nó thực hiện một hành động khiến nó chưa sẵn sàng.
    * Một *preemptible* (ưu tiên) thread có giá trị ưu tiên không âm. Khi nó trở thành thread hiện tại, một thread có thể ưu tiên có thể được thay thế bất kỳ lúc nào nếu một *cooperative* thread hoặc một thread có thể ưu tiên có mức độ ưu tiên cao hơn hoặc bằng nhau trở nên sẵn sàng.

Giá trị ưu tiên ban đầu của thread có thể được thay đổi lên hoặc xuống sau khi thread được bắt đầu. Do đó, một thread ưu tiên có thể trở thành một cooperative thread và ngược lại, bằng cách thay đổi mức độ ưu tiên của nó.

.. admonition:: Ghi chú
   :class: note

   Scheduler không đưa ra quyết định heuristic để sắp xếp lại mức độ ưu tiên của các thread. Ưu tiên của thread chỉ được đặt và thay đổi theo yêu cầu của ứng dụng.

   
Kernel hỗ trợ số lượng mức độ ưu tiên của thread hầu như là không giới hạn. Các tùy chọn cấu hình **CONFIG_NUM_COOP_PRIORITIES** và **CONFIG_NUM_PREEMPT_PRIORITIES** chỉ định số mức độ ưu tiên cho từng loại thread, dẫn đến phạm vi ưu tiên có thể sử dụng được sau đây:

    * **cooperative threads**: từ (- CONFIG_NUM_COOP_PRIORITIES) đến -1
    * **preemptive threads**: từ 0 đến (CONFIG_NUM_PREEMPT_PRIORITIES - 1)

    
.. figure:: imagines/priorities.svg
    :align: center
    :alt: Priority
    :scale: 100%

Ví dụ: cấu hình mức độ ưu tiên cooperative 5 và mức độ ưu tiên ưu tiên preemtive 10 sẽ có kết quả lần lượt là trong phạm vi từ -5 đến -1 và 0 đến 9.


Meta-IRQ Priorities
~~~~~~~~~~~~~~~~~~~

Khi được bật (xem CONFIG_NUM_METAIRQ_PRIORITIES), sẽ có một lớp (subclass) đặc biệt về mức độ ưu tiên cooperative ở đầu cao nhất (thấp nhất về mặt số lượng) của không gian ưu tiên: meta-IRQ threads. Chúng được lên lịch theo mức độ ưu tiên thông thường nhưng cũng có khả năng đặc biệt để ưu tiên tất cả các thread khác (và các meta-IRQ thread khác) ở mức độ ưu tiên thấp hơn, ngay cả khi các thread đó cooperative và/hoặc đã bị khóa bởi scheduler. Tuy nhiên, các thread Meta-IRQ vẫn là các thread và vẫn có thể bị ngắt (gián đoạn) bởi bất kỳ hardware ngắt nào.

Hành vi này thực hiện hành động bỏ chặn một meta-IRQ thread (bằng mọi cách, ví dụ: tạo nó, gọi k_sem_give(), v.v.) tương đương với lệnh gọi hệ thống đồng bộ (synchronous system call) khi được thực hiện bởi một thread có mức độ ưu tiên thấp hơn hoặc giống như ARM “IRQ đang chờ xử lý” khi được thực hiện từ bối cảnh ngắt (interrupt context) thực sự. Mục đích là tính năng này sẽ được sử dụng để triển khai các tính năng xử lý “bottom half” và/hoặc “tasklet” ngắt trong các hệ thống con trình điều khiển. Thread sau khi được đánh thức, sẽ được đảm bảo chạy trước khi CPU hiện tại quay trở lại code ứng dụng.

Không giống như các tính năng tương tự trong các hệ điều hành khác, các meta-IRQ thread là các thread thực sự và chạy trên stack của riêng chúng (phải được phân bổ bình thường), không phải stack ngắt trên mỗi CPU. Việc thiết kế để cho phép sử dụng IRQ stack trên các kiến trúc được hỗ trợ đang chờ xử lý.

Lưu ý rằng vì điều này phá vỡ ràng buộc đối với các cooperative thread của API Zephyr (cụ thể là OS sẽ không lên lịch cho thread khác cho đến khi thread hiện tại cố tình chặn). Do đó chỉ nên sử dụng nó một cách hết sức cẩn thận từ mã ứng dụng. Đây không chỉ đơn giản là những thread có mức độ ưu tiên rất cao mà còn không nên được sử dụng như vậy.


Thread Options
--------------

Kernel hỗ trợ một tập hợp nhỏ các tùy chọn thread cho phép một thread nhận được sự xử lý đặc biệt trong các trường hợp cụ thể. Tập hợp các tùy chọn liên quan đến một thread được chỉ định khi thread được sinh ra.

Một thread không yêu cầu bất kỳ tùy chọn thread nào có giá trị tùy chọn bằng 0. Một thread yêu cầu tùy chọn thread sẽ chỉ định nó theo tên, sử dụng | ký tự làm dấu phân cách nếu cần nhiều tùy chọn (tức là kết hợp các tùy chọn bằng toán tử OR theo bit).



:ref: 'K_ESSENTIAL' và một số thứ lung tung


adsjkfhkfhsahfjkhasdjkdhfjksdhkjfhsjkahfjklhasdjkdhfjkashdjkf

.. _api1:

    K_ESSENTIAL

    system thread that must not abort