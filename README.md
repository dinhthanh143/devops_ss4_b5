# Báo cáo: Khôi phục trạng thái và Đảo ngược commit (Reset vs Revert)

## 1. Mục tiêu
- Thành thạo hai kỹ thuật khắc phục sai sót trong Git: `git reset` (sửa đổi cục bộ) và `git revert` (đảo ngược công cộng an toàn khi làm việc nhóm).
- Hiểu rõ bản chất, sự khác biệt và ngữ cảnh áp dụng của từng phương pháp.
- Đảm bảo tính toàn vẹn của mã nguồn và lịch sử commit theo đúng chuẩn DevOps.

---

## 2. So sánh chi tiết: `git reset` vs `git revert`

| Tiêu chí | `git reset` | `git revert` |
| :--- | :--- | :--- |
| **Bản chất** | Di chuyển con trỏ `HEAD` lùi về commit cũ trong quá khứ, viết lại lịch sử commit. | Tạo ra một **commit mới** có nội dung đối lập hoàn toàn để triệt tiêu thay đổi của commit cần đảo ngược. |
| **Lịch sử commit** | Xóa/ẩn các commit sau mốc reset khỏi nhánh hiện tại. | Bảo toàn nguyên vẹn toàn bộ lịch sử commit, có thêm commit đảo ngược mới. |
| **Phạm vi áp dụng** | **Chỉ dùng cho Local (Cá nhân)** khi commit chưa được đẩy lên Remote. | **Bắt buộc dùng khi làm việc nhóm / Public Branch** (commit đã được push lên Remote). |
| **Rủi ro khi dùng trên Remote** | Rất nguy hiểm (yêu cầu `git push --force`, gây xung đột và hỏng nhánh của đồng đội). | Hoàn toàn an toàn, đồng đội chỉ cần `git pull` bình thường mà không bị xung đột lịch sử. |

---

## 3. Quá trình thực hành & Minh chứng

### 3.1. Trường hợp 1: Sửa đổi cục bộ bằng `git reset` (Không mất code)
- **Tình huống**: Vừa commit lỗi ở local: commit `a3d0519` ("feat: add faulty coupon discount logic").
- **Yêu cầu**: Lùi lại 1 commit, giữ lại code đã sửa đổi trong Working Directory để viết lại cho đúng, không làm mất code (không dùng `--hard`).

**Thực hiện lệnh:**
```bash
git reset HEAD~1
# hoặc: git reset --mixed HEAD~1
```

**Kiểm tra trạng thái (`git status`):**
```bash
git status
```
**Output:**
```plaintext
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   app.js

no changes added to commit (use "git add" and/or "git commit -a")
```
*(Con trỏ HEAD đã lùi về commit trước, file `app.js` được giữ lại ở Working Directory dưới dạng `modified` để lập trình viên chỉnh sửa hoàn thiện trước khi commit lại).*

Sau khi sửa code chuẩn:
```bash
git add app.js
git commit -m "feat: add validated coupon logic"
```

---

### 3.2. Trường hợp 2: Đảo ngược an toàn bằng `git revert` (Làm việc nhóm)
- **Tình huống**: Commit lỗi `ff871cb` ("feat: add unstable crypto payment feature") đã xuất hiện trên nhánh làm việc chung.
- **Yêu cầu**: Đảo ngược thay đổi mà không xóa commit cũ, tạo commit mới an toàn.

**Thực hiện lệnh:**
```bash
git revert HEAD --no-edit
```

**Output:**
```plaintext
[main 9d67cc9] Revert "feat: add unstable crypto payment feature"
 1 file changed, 2 deletions(-)
```

---

## 4. Kết quả kiểm tra lịch sử commit (`git log`)

Chạy lệnh kiểm tra 5 commit gần nhất:
```bash
git log --oneline -n 5
```

**Output:**
```plaintext
9d67cc9 (HEAD -> main) Revert "feat: add unstable crypto payment feature"
ff871cb feat: add unstable crypto payment feature
db73f5f feat: add validated coupon logic
7f47753 feat: initial stable payment system
```

**Đánh giá kết quả**:
1. Commit đảo ngược `9d67cc9` với tiêu đề `Revert "..."` được tạo thành công trên đỉnh nhánh `main`.
2. Toàn bộ lịch sử commit trước đó (`ff871cb`, `db73f5f`, `7f47753`) được bảo toàn 100%, không bị xáo trộn.
