Tích hợp ReSukiSU + Susfs 2.0.0 inline hook cho Kernel Non-GKI


YÊU CẦU:
- Kernel source chưa có ReSukiSU/KernelSU
- Có thể đã có Susfs phiên bản cũ (v1.x)
- Cần tích hợp cả ReSukiSU kernel module và Susfs mới

BƯỚC 1: Thêm ReSukiSU vào kernel tree
1. Di chuyển đến thư mục gốc của kernel
2. Chạy lệnh:
   curl -LSs "https://raw.githubusercontent.com/ReSukiSU/ReSukiSU/main/kernel/setup.sh" | bash

Kết quả: Tạo symlink drivers/kernelsu/ trỏ đến ReSukiSU kernel source

BƯỚC 2: Xóa toàn bộ Susfs cũ nếu có 
1. Xóa các file susfs cũ:
   - fs/susfs.c
   - include/linux/susfs.h
   - include/linux/susfs_def.h

2. Revert các hooks cũ trong kernel files (nếu có)

BƯỚC 3: Apply patch susfs
1. Kiểm tra phiên bản kernel Makefile để chọn susfs patch phù hợp

2. Tải patch, thay 4.19 bằng phiên bản kernel hiện tại nếu khác hoặc clone repo để liệt kê file:
   curl -sL "https://raw.githubusercontent.com/JackA1ltman/NonGKI_Kernel_Build_2nd/refs/heads/mainline/Patches/Patch/susfs_patch_to_4.19.patch" -o susfs_patch.patch

3. Apply patch (có thể có rejects):
   patch -p1 --force < susfs_patch.patch

4. Kiểm tra và xử lý các file .rej:
   - Tìm tất cả file .rej: find . -name "*.rej"
   - Với mỗi file .rej:
     a. Đọc nội dung reject
     b. Kiểm tra xem code đã được apply chưa trong file gốc (grep tìm keyword từ reject)
     c. Nếu đã có → xóa file .rej
     d. Nếu chưa có → apply thủ công bằng cách thêm code vào vị trí thích hợp

5. Xóa các file .orig backup:
   find . -name "*.orig" -delete

BƯỚC 4: Apply susfs_inline_hook_patches.sh
1. Tải script:
   curl -sL "https://raw.githubusercontent.com/JackA1ltman/NonGKI_Kernel_Build_2nd/refs/heads/mainline/Patches/susfs_inline_hook_patches.sh" -o susfs_inline.sh

2. Chạy script:
   bash susfs_inline.sh

Lưu ý: Script sẽ tự động skip các file đã có KernelSU hooks

BƯỚC 5: Xác nhận kết quả
1. Kiểm tra version susfs:
   grep "SUSFS_VERSION" include/linux/susfs.h

2. Kiểm tra các file đã được patch:
   - fs/exec.c, fs/open.c, fs/read_write.c, fs/stat.c
   - kernel/reboot.c, kernel/sys.c
   - security/selinux/hooks.c

3. Kiểm tra không còn reject files:
   find . -name "*.rej"

4. Kiểm tra các file hỗ trợ có sẵn trong kernel source:
   - Nếu đã thêm ReSukiSU bằng curl: Kiểm tra thư mục drivers/kernelsu/kernel/tools/
   - Nếu đã thêm KernelSU thủ công: Kiểm tra thư mục KernelSU/kernel/tools/

XỬ LÝ SỰ CỐ THƯỜNG GẶP

1. Patch bị conflict với code hiện tại:
   → Dùng patch --force hoặc xóa hoàn toàn susfs cũ rồi apply lại

2. Reject files không apply được:
   → Đọc từng file .rej, tìm trong code gốc bằng grep, nếu đã có code tương tự thì xóa reject

3. Inline hooks bị skip do đã có KSU:
   → Bình thường, code KSU đã có sẵn trong ReSukiSU

4. Biên dịch lỗi thiếu symbols:
   → Kiểm tra các extern declarations đã được thêm vào chưa

SAU KHI HOÀN TẤT
- Agent kiểm tra trong kernel tree các script build (build.sh, build_kernel.sh, compile.sh):
  1. Nếu tìm thấy script build → đọc nội dung để xác định file defconfig được sử dụng (ví dụ: KERNEL_DEFCONFIG=...)
  2. Mở file defconfig tương ứng trong arch/arm64/configs/
  3. Thêm các dòng sau vào defconfig:
     CONFIG_KSU=y
     CONFIG_KSU_SUSFS=y

- Nếu không tìm thấy script build nào:
  1. Agent liệt kê tất cả file config trong arch/arm64/configs/ (bao gồm cả thư mục con)
  2. Người dùng chọn file defconfig phù hợp
  3. Agent thêm các dòng sau vào defconfig đã chọn:
     CONFIG_KSU=y
     CONFIG_KSU_SUSFS=y
