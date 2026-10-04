# Bài 5: Khôi phục trạng thái và Đảo ngược commit (Reset vs Revert)

## 1. Mục tiêu

Bài thực hành nhằm phân biệt và sử dụng hai cơ chế `git reset` và `git revert` để xử lý các commit có vấn đề.

* `git reset`: sử dụng khi cần thay đổi lịch sử Git cục bộ.
* `git revert`: sử dụng khi cần đảo ngược một commit đã được chia sẻ trên remote mà không viết lại lịch sử.

---

## 2. Trường hợp 1: Sử dụng git reset

### Bối cảnh

Một commit gần nhất chứa mã nguồn bị lỗi. Tôi muốn đưa HEAD lùi lại 1 commit nhưng vẫn giữ nguyên các thay đổi trong Working Directory để tiếp tục chỉnh sửa code.

### Commit ban đầu

Kiểm tra lịch sử:

```bash
git log --oneline -n 5
```

Ví dụ:

```text
a1b2c3d (HEAD -> main) feat: add buggy code
9f8e7d6 previous commit
```

### Thực hiện reset

Sử dụng:

```bash
git reset HEAD~1
```

Lệnh này tương đương với:

```bash
git reset --mixed HEAD~1
```

Không sử dụng `git reset --hard` vì yêu cầu của bài là không được làm mất mã nguồn hiện hành.

### Kiểm tra

```bash
git status
```

Kết quả mong đợi:

```text
Changes not staged for commit:
  modified:   bug.txt
```

Như vậy:

* HEAD đã lùi lại 1 commit.
* Commit lỗi không còn là commit hiện tại.
* Các thay đổi của commit lỗi vẫn được giữ lại trong Working Directory.
* File được hiển thị ở trạng thái `Modified`.

---

## 3. Trường hợp 2: Sử dụng git revert

### Bối cảnh

Một commit chứa mã nguồn lỗi đã được chia sẻ trên remote hoặc nằm trên nhánh dùng chung.

Trong trường hợp này không nên sử dụng `git reset` để viết lại lịch sử vì có thể gây ảnh hưởng đến các thành viên khác trong nhóm.

### Xác định commit cần đảo ngược

Kiểm tra lịch sử:

```bash
git log --oneline -n 5
```

Ví dụ:

```text
a1b2c3d feat: add buggy code
9f8e7d6 previous commit
```

Thực hiện:

```bash
git revert a1b2c3d
```

Trong đó `a1b2c3d` là commit cần đảo ngược.

Git sẽ tạo một commit mới để hoàn tác các thay đổi của commit được chỉ định.

### Kiểm tra lịch sử

```bash
git log --oneline -n 5
```

Kết quả mong đợi:

```text
e4f5g6h (HEAD -> main) Revert "feat: add buggy code"
a1b2c3d feat: add buggy code
9f8e7d6 previous commit
```

Commit ban đầu vẫn tồn tại trong lịch sử và một commit `Revert ...` mới được tạo ở phía trên.

---

## 4. Sự khác biệt giữa Reset và Revert

### Git Reset

`git reset` di chuyển HEAD về một commit khác và có thể thay đổi lịch sử của branch.

Ví dụ:

```text
Trước:

A ---- B ---- C (HEAD)

Sau reset:

A ---- B (HEAD)
```

Với:

```bash
git reset HEAD~1
```

các thay đổi của commit C vẫn được giữ trong Working Directory dưới dạng chưa staged.

Reset phù hợp hơn khi làm việc cá nhân trên lịch sử cục bộ, đặc biệt khi commit chưa được push lên remote.

### Git Revert

`git revert` không xóa commit cũ mà tạo ra một commit mới để đảo ngược thay đổi.

Ví dụ:

```text
Trước:

A ---- B ---- C (HEAD)

Sau revert:

A ---- B ---- C ---- D (HEAD)
                    ^
                 Revert C
```

Commit C vẫn tồn tại trong lịch sử, còn D là commit mới dùng để hoàn tác C.

Revert phù hợp khi commit đã được push lên remote hoặc đang nằm trên branch dùng chung với các thành viên khác.

---

## 5. So sánh

| Tiêu chí                      | Reset        | Revert |
| ----------------------------- | ------------ | ------ |
| Di chuyển HEAD                | Có           | Không  |
| Tạo commit mới                | Không        | Có     |
| Viết lại lịch sử branch       | Có thể       | Không  |
| Giữ commit cũ trong lịch sử   | Có thể không | Có     |
| Phù hợp với local branch      | Có           | Có     |
| Phù hợp với branch dùng chung | Không nên    | Nên    |
| An toàn sau khi đã push       | Không nên    | Có     |

---

## 6. Kết luận

Qua bài thực hành:

* `git reset HEAD~1` được sử dụng để đưa HEAD lùi lại một commit nhưng vẫn giữ thay đổi trong Working Directory.
* Không sử dụng `git reset --hard` vì có thể làm mất các thay đổi hiện hành.
* `git revert <commit-id>` tạo một commit mới để đảo ngược một commit cũ.
* Reset phù hợp hơn với việc chỉnh sửa lịch sử cục bộ chưa chia sẻ.
* Revert an toàn hơn khi làm việc nhóm và với các commit đã được push lên remote.

Lệnh kiểm tra cuối cùng:

```bash
git log --oneline -n 5
```

Kết quả cần có commit mới bắt đầu bằng:

```text
Revert ...
```
