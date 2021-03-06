Bài viết được dịch từ http://phrack.org/issues/49/14.html#article
I. Cấu trúc bộ nhớ 

Để hiểu những gì bộ đệm của stack đầu tiên chúng ta phải hiểu quá trinh này được tổ chức như thế nào trong bộ nhớ. Quá trình được chia làm 3 phần. Text, Data và Stack. Chúng ta sẽ tập trung về phần Stack, nhưng trước tiên cần có một cái nhìn tổng quan về các khu vực nhỏ khác theo thứ tự.
  
  Các khu vực văn bản được cố định bởi chương trình và bao gồm cả code, chỉ đọc dữ liệu. Khu vực này tương ứng với phần văn bản của các tệp tin được thực thi. Khu vực này được đánh dấu chỉ đọc và bất kỳ sự xâm phạm để ghi nó sẽ dẫn đến kết quả trong một segmentation violation.

  Khu vực chứa data được khởi tạo và chưa được khởi tạo. Các biến tĩnh được lưu trữ ở khu vực này. Vùng data tương ứng với phần data-bss của file thực thi. Kích thước của nó có thể thay đổi khi hệ thống gọi brk(2). Nếu việc mở rộng data-bss hoặc người sử dụng hết stack bộ nhớ có sẵn, quá trình này sẽ bị chặn và được dời lại để chạy với không gian bộ nhớ lớn hơn. Bộ nhớ mới sẽ được thêm vào giữa data và stack segments.

//////////////////////////////////////////////////////////////////////////////////////////////////////////////

Stack là gì ?

Một stack là một kiểu dữ liệu trừu tượng thường được sử dụng trong khoa học máy tính. Một stack của đối tượng có đặc tính mà các đối tượng cuối cùng đặt lên trên stack sẽ là đối tượng đầu tiên được lấy ra, loại bỏ. Đặc tính được gọi chung là vào sau ra trước hoặc LIFO.

Một số hoạt động được xác định trên stack và 2 cái quan trọng nhất là push và pop. PUSH là thêm một phần tử vào đỉnh stack.  POP thì ngược lại, giảm kích thước của stack bằng một cách loại bỏ một phần tử cuối cùng tại đỉnh stack.

Tại sao chúng ta sử dụng stack ?

Các stack cũng được sử dụng tự động phân phát các biến cục bộ sử dụng trong funcitons, để truyền tham số funciton và trả về một giá trị của funciton.
 
Vùng Stack

Một stack là một khối liền kề của bộ nhớ chứa dữ liệu. Một thanh ghi được gọi khi con trỏ ngăn xếp (SP) trỏ đến đỉnh stack. Phần dưới cùng của stack sẽ là một địa chỉ được cố định. Kích thước của nó tự động điểu chỉnh bởi kernel trong lúc chạy. CPU thực hiện các hướng dẫn để PUSH và POP lên stack. Các stack gồm các stack frames luận lí để PUSH khi gọi một function và POP khi trả về, bao gồm giá trị của con trỏ tại thời điểm khi function được gọi.

Tùy thuộc vào các việc thực hiện mà stack sẽ xuống ( về phía các địa chỉ bộ nhớ thấp hơn) hoặc tăng.  Trong ví dụ dưới đây, chúng tôi sẽ sử dụng stack để giảm xuống địa chỉ bộ nhớ thấp hơn. Đây là cách ngăn xếp phát triển trên nhiều máy tính bao gồm Intel, bộ vi xử lý Motorola, SPARC và MIPS. Con trỏ stack (SP) cũng phụ thuộc vào việc thực hiện. Nó có thể đến vị trí cuối cùng của stack hoặc đến địa chỉ có sẵn không sử dụng tiếp theo. Để thảo luận, chúng tôi sẽ chiếm lấy con trỏ đến địa chỉ cuối của stack.

Ngoài các con trỏ stack mà điểm đến của nó là đỉnh stack ( địa chỉ thấp nhất), nó cũng thích hợp cho việc frame pointer  (FP)  chỉ vào một vị trí cố định trong frame. Một số text cũng được xem như một local base pointer (LB).  Về nguyên tắc, các biến cục bộ cũng có thể tham chiếu bằng cách địa chỉ offset từ SP. Tuy nhiên, PUSH lên đỉnh stack và POP từ stack thì offset cũng thay đổi. Mặc dù trong một số trường hợp trình biên dịch có thể theo dõi số lượng từ stack, từ đó điểu chính các offset, trong một vài trường hợp có thể không và trong mọi trường hợp administration là cần thiết. Hơn nữa trên một số máy, chẳng hạn như bộ xử lí Intel, truy cập một biến cách xa từ SP đòi hỏi phải có nhiều hướng dẫn. 

Do đó, nhiều trình biên dịch sử dụng một thanh ghi thứ hai, FP, để tham chiếu đến các biến cục bộ và các tham số bởi vì cách biệt ra từ FP không làm thay đổi PUSH và POP.
Trên các CPU Intel, BP (EBP) được sử dụng cho mục đích này. Trên các CPU Motorola, sẽ làm bất kì thanh ghi nào ngoại trừ A7 ( con trỏ stack) . Bởi vì cách mà chúng tôi phát triển stack, các tham số offset lớn hơn số không và biến cục bộ không nhận offset từ FP.

Điều đầu tiên của một thủ tục phải làm khi được gọi là lưu ở FP trước (do đó, nó có thể được phục hồi ở phần thoát của thủ tục). Sau đó nó sao chép SP vào FP để tạo FP mới và sự tăng lên của SP từ không gian lưu trữ cho các biến cục bộ. Những code này được là các thủ tục prolog. Khi các thủ tục thoát, stack sẽ được làm trống một lần nữa.

Chúng ta hãy xem những gì diễn ra ở stack ở một ví dụ đơn giản:

 example1.c :
---------------------------------------------------------------------------------------------------------
void function(int a, int b, int c) {
   char buffer1[5];
   char buffer2[10];
}

void main() {
  function(1,2,3);
}
----------------------------------------------------------------
Để hiểu được chương trình sẽ làm gì khi gọi function, chúng ta sẽ biên dịch nó với gcc sử dụng lựa chọn –S để tạo ra code assembly:

$ gcc -S -o example1.s example1.c

Bằng cách nhìn và ngôn ngữ assembly chúng ta sẽ thấy chương trình khi gọi một hàm sẽ được dịch ra như dưới đây: 
        pushl $3
        pushl $2
        pushl $1
        call function

Đoạn ở trên này sẽ đẩy 3 tham số của một function từ phải qua trái vào stack và gọi function(). Các hướng dẫn call sẽ đẩy con trỏ IP và stack. Chúng tôi sẽ gọi IP là nơi lưu trữ địa chỉ được trả về (RET). Đây sẽ là điều đầu tiên được thực hiện thủ tục prolog trong một function: 

       pushl %ebp
       movl %esp,%ebp
       subl $20,%esp

Điều này sẽ đẩy EBP (frame pointer) và đỉnh stack. Nó sao chép EBP vào SP, tạo ra một FP mới. Chúng tôi sẽ gọi FP là nơi lưu trữ con trỏ SFP (con trỏ chứa địa chỉ bắt đầu của stack). Sau đó nó sẽ phân chia không gian cho các biến cục bộ bằng cách trừ đi kích thước của ESP. Chúng ta phải nhớ, bộ nhớ chỉ có thể được giải quyết trong nhiều phần của kích thước word. Một word trong trường hợp của chúng ta sẽ chiếm 4 bytes hoặc 32bit, 1 byte = 8 bit. Vì vậy, 5 bytes của bộ đệm thực sự sẽ mất 8 bytes ( 2 words) của bộ nhớ và 10 bytes của bộ đệm sẽ mất 12 bytes ( 3 words) của bộ nhớ. Đó là lí do tại sao SP lại trừ đi 20. Với những điều này, stack sẽ như thế này khi function được gọi ( mỗi không gian lưu trữ đại diện cho 1 byte):

bottom of                                                            top of
memory                                                               memory
               buffer2     buffer1     sfp    ret       a      b     c
<------   [            ][                ][      ][       ][       ][      ][     ]
	   
top of                                                            	bottom of
stack                                                                        stack

			
			Tràn bộ đệm

Một lỗi tràn bộ đệm là kết quả của việc dữ liệu đưa vào bộ đệm nhiều hơn việc nó có thể xử lý. Làm thế nào để thường xuyên tìm thấy các lỗi của trường trình có thể thực thi các mã này. Hãy xem xét ví dụ dưới đây : 






example2.c
------------------------------------------------------------------------------
void function(char *str) {
   char buffer[16];

   strcpy(buffer,str);
}

void main() {
  char large_string[256];
  int i;

  for( i = 0; i < 255; i++)
    large_string[i] = 'A';

  function(large_string);
}

Chương trình trên có một lỗi tràn bộ đệm tiêu biểu. Hàm kiểm tra sao chép một chuỗi mà không kiểm tra giới hạn được sử dụng strcpy() thay vì strncpy(). Nếu bạn chạy chương trình này, bạn sẽ nhận được một segmentation violation. Xem những gì diễn ra ở stack khi chúng ta gọi function.

bottom of                                                            top of
memory                                                               memory
                      buffer            sfp   ret   *str
<------          [                ][          ][     ][     ]

top of                                                            bottom of
stack                                                                 stack


Chuyện gì đang xảy ra ở đây ? Tại sao chúng ta lại có một segmentation violation ? Rất đơn giản, strcpy() sẽ sao chép nội dung của *str (larger_string[]) trong buffer[] cho đến khi một kí tự NULL được tìm thấy. Như chúng ta thấy buffer[] nhỏ hơn rất nhiều so với *str, buffer[] chỉ có 16 byte và chúng tôi cố gắng ghi đè nó với 256 byte. Điều này có nghĩa rằng tất cả 250 byte còn lại của stack đều sẽ bị ghi đè lên, bao gồm cả SFP, RET và *str. Chúng tôi sẽ điền vào các chuỗi bằng kí tự A, nó sẽ mang giá trị hex là 0x41. Điều này có nghĩa rằng địa chỉ trả về sẽ là 0x41414141. Đây là bên ngoài của không gian địa chỉ tiến trình. Nó cũng là lí do tại sao khi function trả về và cố gắng đọc tiếp từ những gì của đại chỉ sẽ có một segmentation violation. 

Vì vậy một lỗi buffer overflow cho phép chúng ta thay đổi địa chỉ trả về của một function. Bằng cách này chúng ta có thể thay đổi các luồng thực thi của chương trình. Quay lại ví dụ đầu tiên của chúng ta và xem lại stack có cái gì: 
bottom of                                                            top of
memory                                                               memory
           buffer2       buffer1   sfp   ret   a     b     c
<------   [            ][        ][    ][    ][    ][    ][    ]

top of                                                            bottom of
stack                                                                 stack


Cố gắng sửa đổi ở ví dụ đầu tiên của chúng ta để nó ghi đè lên địa chỉ trả về và làm thế nào để thực thi các mã tùy ý. Chỉ có trước buffer1[] trên stack là SFP và trước nó, địa chỉ trả về. Đó là vượt qua được 4 byte cuối của buffer1[]. Nhưng nhớ rằng buffer1[] thực sự là 2 word vì thế nó cũng như 8 byte. Vì vậy địa chỉ trả về là 12 byte bắt đầu từ buffer1[]. Chúng tôi sẽ sửa đổi giá trị trả về bằng lệnh gán ‘x = 1;’ sau khi gọi function. Làm vậy để chúng tôi thêm 8 byte vào địa chỉ trả về. 

example3.c:
------------------------------------------------------------------------------
void function(int a, int b, int c) {
   char buffer1[5];
   char buffer2[10];
   int *ret;

   ret = buffer1 + 12;
   (*ret) += 8;
}

void main() {
  int x;

  x = 0;
  function(1,2,3);
  x = 1;
  printf("%d\n",x);
}
------------------------------------------------------------------------------

Những gì đã làm ở trên là cộng thêm 12 vào địa chỉ của buffer1[]. Địa chỉ mới này là nơi địa chỉ trả về được lưu trữ. Chúng tôi muốn bỏ qua việc gọi printf(). Làm thế nào để chúng ta cộng thêm 8 vào địa chỉ trả về. Chúng tôi sủ dụng thử một giá trị ban đầu (ví dụ 1), biên dịch chương trình sau đó bắt đầu với GDB. 

[aleph1]$ gdb example3
GDB is free software and you are welcome to distribute copies of it
 under certain conditions; type "show copying" to see the conditions.
There is absolutely no warranty for GDB; type "show warranty" for details.
GDB 4.15 (i586-unknown-linux), Copyright 1995 Free Software Foundation, Inc...
(no debugging symbols found)...
(gdb) disassemble main
Dump of assembler code for function main:
0x8000490 <main>:       pushl  %ebp
0x8000491 <main+1>:     movl   %esp,%ebp
0x8000493 <main+3>:     subl   $0x4,%esp
0x8000496 <main+6>:     movl   $0x0,0xfffffffc(%ebp)
0x800049d <main+13>:    pushl  $0x3
0x800049f <main+15>:    pushl  $0x2
0x80004a1 <main+17>:    pushl  $0x1
0x80004a3 <main+19>:    call   0x8000470 <function>
0x80004a8 <main+24>:    addl   $0xc,%esp
0x80004ab <main+27>:    movl   $0x1,0xfffffffc(%ebp)
0x80004b2 <main+34>:    movl   0xfffffffc(%ebp),%eax
0x80004b5 <main+37>:    pushl  %eax
0x80004b6 <main+38>:    pushl  $0x80004f8
0x80004bb <main+43>:    call   0x8000378 <printf>
0x80004c0 <main+48>:    addl   $0x8,%esp
0x80004c3 <main+51>:    movl   %ebp,%esp
0x80004c5 <main+53>:    popl   %ebp
0x80004c6 <main+54>:    ret
0x80004c7 <main+55>:    nop


Chúng ta có thể có thấy rằng khi gọi function thì RET sẽ là 0x8004a8 và chúng ta muốn bỏ qua việc tại 0x80004ab. Ở hướng dẫn tiếp theo chúng ta muốn thực thi mã tại 0x80004b2. Một chút về toán học cho chúng ta biết khoảng cách là 8 byte.
