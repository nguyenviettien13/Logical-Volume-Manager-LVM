
# Logical Volume Manager (LVM)

### Mục lục

[1. Giới thiệu về Logical Volume Manager (LVM)](#Gioithieu)
- [1.1 LVM là gì](#LVM)
- [1.2 Vai trò của LVM](#Vaitro)
- [1.3 Các thành phần trong LVM](#Thanhphan)

[2. Hướng dẫn sử dụng LVM](#Huongdan)
- [2.1 Chuẩn bị](#Chuanbi)
- [2.2 Tạo Logical Volume trên LVM](#Taolvm)
- [2.3 Thay đổi dung lượng Logical Volume trên LVM](#Thaydoilv)
- [2.4 Thay đổi dung lượng Volume Group trên LVM](#Thaydoivg)
- [2.5 Xóa Logical Volume, Volume Group, Physical Volume](#Xoa)

[3. Tổng kết](#Tongket)

<a name="Gioithieu"></a>
## 1. Giới thiệu về Logical Volume Manager (LVM)

<a name="LVM"></a>
### 1.1 LVM là gì

Logical Volume Manager (LVM): là phương pháp cho phép ấn định không gian đĩa cứng thành những logical Volume khiến cho việc thay đổi kích thước trở nên dễ dàng hơn (so với partition). Với kỹ thuật Logical Volume Manager (LVM) bạn có thể thay đổi kích thước mà không cần phải sửa lại table của OS. Điều này thật hữu ich với những trường hợp bạn đã sử dụng hết phần bộ nhớ còn trống của partition và muốn mở rộng dung lượng của nó

<a name="Vaitro"></a>
### 1.2 Vai trò của LVM

LVM là kỹ thuật quản lý việc thay đổi kích thước lưu trữ của ổ cứng

<ul>
<li>Không để hệ thống bị gián đoạn hoạt động</li>

<li>Không làm hỏng dịch vụ</li>

<li>Có thể kết hợp Hot Swapping (thao tác thay thế nóng các thành phần bên trong máy tính)</li>
</ul>

<a name="Thanhphan"></a>
### 1.3 Các thành phần trong LVM

**Mô hình các thành phần trong LVM**

<img src="http://i.imgur.com/UaTatub.png">

**Hard drives – Drives**

Thiết bị lưu trữ dữ liệu, ví dụ như trong linux nó là `/dev/sda`

**Partition**

Partitions là các phân vùng của Hard drives, mỗi Hard drives có 4 partition, trong đó partition bao gồm 2 loại là primary partition và extended partition

- **Primary partition:**
<ul>
<li>Phân vùng chính, có thể khởi động</li>
<li>Mỗi đĩa cứng có thể có tối đa 4 phân vùng này</li>
</ul>

- **Extended partition:**
<ul>
<li>Phân vùng mở rộng, có thể tạo những vùng luân lý</li>
</ul>

**Physical Volumes**

Là một cách gọi khác của partition trong kỹ thuật LVM, nó là những thành phần cơ bản được sử dụng bởi LVM. Một Physical Volume không thể mở rộng ra ngoài phạm vi một ổ đĩa.

Chúng ta có thể kết hợp nhiều Physical Volume thành Volume Groups

**Volume Group**

Nhiều Physical Volume trên những ổ đĩa khác nhau được kết hợp lại thành một Volume Group 

<img src="http://i.imgur.com/jIcqtMS.png">

Volume Group được sử dụng để tạo ra các Logical Volume, trong đó người dùng có thể tạo, thay đổi kích thước, lưu trữ, gỡ bỏ và sử dụng.

*Một điểm cần lưu ý là boot loader không thể đọc /boot khi nó nằm trên Logical Volume Group. Do đó không thể sử dụng kỹ thuật LVM với /boot mount point*

**Logical Volume**

Volume Group được chia nhỏ thành nhiều Logical Volume, mỗi Logical Volume có ý nghĩa tương tự như partition. Nó được dùng cho các mount point và được format với những định dạng khác nhau như ext2, ext3, ext4,...

<img src="http://i.imgur.com/SbQZKnu.png">

Khi dung lượng của Logical Volume được sử dụng hết ta có thể đưa thêm ổ đĩa mới bổ sung cho Volume Group và do đó tăng được dung lượng của Logical Volume

Ví dụ bạn có 4 ổ đĩa mỗi ổ 5GB khi bạn kết hợp nó lại thành 1 volume group 20GB, và bạn có thể tạo ra 2 logical volumes mỗi disk 10GB

**File Systems**

<ul>
<li>Tổ chức và kiểm soát các tập tin</li>
 
<li>Được lưu trữ trên ổ đĩa cho phép truy cập nhanh chóng và an toàn</li>

<li>Sắp xếp dữ liệu trên đĩa cứng máy tính</li>

<li>Quản lý vị trí vật lý của mọi thành phần dữ liệu</li>
</ul>

<a name="Huongdan"></a>
## 2. Hướng dẫn sử dụng LVM

<a name="Chuanbi"></a>
### 2.1 Chuẩn bị

<ul>
<li>Tạo máy ảo trên vmware Workstation cài hệ điều hành ubuntu server 12.04</li>

<li>Add thêm một số ổ cứng vào máy ảo</li>
</ul>

<img src="http://i.imgur.com/5gdAnRr.png">

<a name="Taolvm"></a>
### 2.2 Tạo Logical Volume trên LVM

**B1. Kiểm tra các Hard Drives có trên hệ thống**

Bạn có thể kiểm tra xem có những Hard Drives nào trên hệ thống bằng cách sử dụng câu lệnh `lsblk`

`# lsblk`

<img src="http://i.imgur.com/6LZcOvV.png">

Trong đó sdb, sdc, sdd, sde là các Hard Drives mà mình mới thêm vào

**B2. Tạo Partition**

Từ các Hard Drives trên hệ thống, bạn tạo các partition. Ở đây, từ sdb, mình tạo các partition bằng cách sử dụng lệnh sau `fdisk /dev/sdb`

<img src="http://i.imgur.com/ezJlmge.png">

<li>Trong đó bạn chọn `n` để bắt đầu tạo partition</li>

<li>Bạn chọn `p` để tạo partition primary</li>

<li>Bạn chọn `1` để tạo partition primary 1</li>

<li>Tại `First sector (2048-20971519, default 2048)` bạn để mặc định</li>

<li>Tại `Last sector, +sectors or +size{K,M,G} (2048-20971519, default 20971519)` bạn chọn `+1G` để partition bạn tạo ra có dung lượng 1 G</li>

<li>Bạn chọn `w` để lưu lại và thoát.</li>

Tiếp theo bạn thay đổi định dạng của partition vừa mới tạo thành LVM

<img src="http://i.imgur.com/Ih9riPU.png">

<li>Bạn chọn `t` để thay đổi định dạng partition</li>

<li>Bạn chọn `8e` để đổi thành LVM</li>

Tương tự, bạn tạo thêm các partition primary từ sdb

<img src="http://i.imgur.com/awFqKgt.png">

Tạo các partition primary từ `sdc` bằng lệnh `fdisk /dev/sdc`

<img src="http://i.imgur.com/O4SEOBE.png">

**B3. Tạo Physical Volume**

Tạo các Physical Volume là `/dev/sdb1` và `/dev/sdc1` bằng các lệnh sau:

`# pvcreate /dev/sdb1`

`# pvcreate /dev/sdc1`

Bạn có thể kiểm tra các Physical Volume bằng câu lệnh `pvs` hoặc có thể sử dụng lệnh `pvdisplay`

<img src="http://i.imgur.com/BxM2Asn.png">

**B4. Tạo Volume Group**

Tiếp theo, mình sẽ nhóm các Physical Volume thành 1 Volume Group bằng cách sử dụng câu lệnh sau:

`# vgcreate vg-demo1 /dev/sdb1 /dev/sdc1`

Trong đó `vg-demo1` là tên của Volume Group

Có thể sử dụng câu lệnh sau để kiểm tra lại các Volume Group đã tạo

`# vgs`

`# vgdisplay`

<img src="http://i.imgur.com/UYWldHO.png">

**B5. Tạo Logical Volume**

Từ một Volume Group, chúng ta có thể tạo ra các Logical Volume bằng cách sử dụng lệnh sau:

`# lvcreate -L 1G -n lv-demo1 vg-demo1`

-L: Chỉ ra dung lượng của logical volume

-n: Chỉ ra tên của logical volume

Trong đó `lv-demo1` là tên Logical Volume, `vg-demo1` là Volume Group mà mình vừa tạo ở bước trước

*Lưu ý là chúng ta có thể tạo nhiều Logical Volume từ 1 Volume Group*

Có thể sử dụng câu lệnh sau để kiểm tra lại các Logical Volume đã tạo

`# lvs`

`# lvdisplay`

<img src="http://i.imgur.com/ApEeeWr.png">

**B6. Định dạng Logical Volume**

Để format các Logical Volume thành các định dạng như ext2, ext3, ext4, ta có thể làm như sau:

`# mkfs -t ext4 /dev/vg-demo1/lv-demo1`

<img src="http://i.imgur.com/Ouc5T4P.png">

**B7. Mount và sử dụng**

Trong bài lab này, mình sẽ tạo ra một thư mục để mount Logical Volume đã tạo vào thư mục đó

`# mkdir demo1`

Tiến hành mount logical volume `lv-demo1` vào thư mục `demo1` như sau:

`# mount /dev/vg-demo1/lv-demo1 demo1`

Kiểm tra lại dung lượng của thư mục đã được mount:

`# df -h`

<img src="http://i.imgur.com/rdzFy1C.png">

<a name="Thaydoilv"></a>
### 2.3 Thay đổi dung lượng Logical Volume trên LVM

Ở phần trước, mình đã tiến hành tạo Logical Volume trong LVM. Ở phần này, chúng ta sẽ tìm hiểu làm thế nào để có thể thay đổi dung lượng của 1 Logical Volume đã được tạo ở phần trước.

Trước khi thay đổi dung lượng, các bạn cần phải kiểm tra các thông tin hiện có:

`# vgs`

`# lvs`

`# pvs`

<img src="http://i.imgur.com/vvTdiJk.png">

Ở đây, mình đã tạo được Logical Volume là lv-demo1, và giả sử Logical Volume này dung lượng đã đầy và chúng ta cần tăng kích thước của nó.

Logical Volume này thuộc Volume Group vg-demo1, để tăng kích thước, bước đầu tiên phải kiểm tra xem Volume Group còn dư dung lượng để kéo giãn Logical Volume không. Logical Volume thuộc 1 Volume Group nhất định, Volume Group đã cấp phát hết thì Logical Volume cũng không thể tăng dung lượng được. Để kiểm tra, ta dùng lệnh sau:

`# vgdisplay`

<img src="http://i.imgur.com/b26PAiV.png">

Volume Group ở đây vẫn còn dung lượng để cấp phát, ta có thể nhận thấy điều này qua 2 trường thông tin là `VG Status  resizable` và `Free PE / Size  510 / 1.99 GiB` với dung lượng Free là 510*4 = 2040 Mb

Để tăng kích thước Logical Volume ta sử dụng câu lệnh sau:

`# lvextend -L +50M /dev/vg-demo1/lv-demo1`

Với `-L` là tùy chọn để tăng kích thước

<img src="http://i.imgur.com/ZaKf17j.png">

Kiểm tra lại bằng cách dùng lệnh `# lvs`

<img src="http://i.imgur.com/aLTcqin.png">

Sau khi tăng kích thước cho Logical Volume thì Logical Volume đã được tăng nhưng file system trên volume này vẫn chưa thay đổi, bạn phải sử dụng lệnh sau để thay đổi:

`# resize2fs /dev/vg-demo1/lv-demo1`

<img src="http://i.imgur.com/hpaTvwd.png">

Để giảm kích thước của Logical Volume, trước hết các bạn phải umount Logical Volume mà mình muốn giảm

`# umount /dev/vg-demo1/lv-demo1`

Tiến hành giảm kích thước của Logical Volume

`# lvreduce -L 20M /dev/vg-demo1/lv-demo1`

Sau đó tiến hành format lại Logical Volume

`# mkfs.ext4 /dev/vg-demo1/lv-demo1`

Cuối cùng là mount lại Logical Volume

`# mount /dev/vg-demo1/lv-demo1 demo1`

Kiểm tra kết quả ta được như sau:

<img src="http://i.imgur.com/1969nvB.png">

<a name="Thaydoivg"></a>
## 2.4 Thay đổi dung lượng Volume Group trên LVM

Ở phần trước mình có thể tăng kích thước của Logical Volume nhưng với điều kiện Volume Group của Logical Volume đó còn dung lượng. Phần này chúng ta sẽ tìm hiểu xem làm thế nào có thể mở rộng thêm kích thước của Volume Group cũng như thu hồi dung lượng của nó.

Việc thay đổi kích thước của Volume Group chính là việc nhóm thêm Physical Volume hay thu hồi Physical Volume ra khỏi Volume Group

Trước tiên, các bạn cần kiểm tra lại các partition và Volume Group

`# vgs`

`# lsblk`

<img src="http://i.imgur.com/6f70rvP.png">

Tiếp theo, nhóm thêm 1 partition vào Volume Group như sau:

`# vgextend /dev/vg-demo1 /dev/sdb3`

<img src="http://i.imgur.com/o80NZHx.png">

Ở đây, muốn nhóm vào Volume Group phải là Physical Volume nên hệ thống đã tự động tạo cho mình Physical Volume và nhóm vào Volume Group.

Chúng ta có thể cắt 1 Physical Volume ra khỏi Volume Group như sau:

`# vgreduce /dev/vg-demo1 /dev/sdb3`

<img src="http://i.imgur.com/5CoUoTy.png">

<a name="Xoa"></a>
## 2.5 Xóa Logical Volume, Volume Group, Physical Volume

**Xóa Logical Volumes**

Trước tiên ta phải Umount Logical Volume

`# umount /dev/vg-demo1/lv-demo1`

Sau đó tiến hành xóa Logical Volume bằng câu lệnh sau:

`# lvremove /dev/vg-demo1/lv-demo1`

Ta kiểm tra lại kết quả

<img src="http://i.imgur.com/bKyhNaF.png">

**Xóa Volume Group**

Trước khi xóa Volume Group, chúng ta phải xóa Logical Volume 

Xóa Volume Group bằng cách sử dụng lệnh sau:

`# vgremove /dev/vg-demo1`

<img src="http://i.imgur.com/o3Dc00L.png">

**Xóa Physical Volume**

Cuối cùng là xóa Physical Volume:

`#  pvremove /dev/sdb3`

<img src="http://i.imgur.com/4KrrO9u.png">

Vậy là mình đã hoàn thành một bài lab đơn giản về LVM.

<a name="Tongket"></a>
## 3. Tổng kết

Bài viết trên mình đã tổng hợp lại nhũng kiến thức cơ bản trong quá trình mình tìm hiểu về LVM, hy vọng nó có ích cho các bạn. 

Bài viết đang có nhiều sai sót và chưa đầy đủ, mong các bạn góp ý cho mình để mình có thể hoàn thiện hơn.

Một số tài liệu tham khảo:

http://www.tecmint.com/extend-and-reduce-lvms-in-linux/

http://www.tecmint.com/manage-multiple-lvm-disks-using-striping-io/

http://www.tecmint.com/create-lvm-storage-in-linux/

http://www.markus-gattol.name/ws/lvm.html#sec29

