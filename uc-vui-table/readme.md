Dưới đây là nội dung tệp `README.md` được định dạng chuẩn chuyên nghiệp cho GitHub dựa trên các thông tin bạn cung cấp.

---

# uc-vui-table

`uc-vui-table` là một component bảng hiệu năng cao được tối ưu hóa với **Virtual Scroll**, hỗ trợ chọn nhiều dòng (checkbox), tùy chỉnh hiển thị linh hoạt qua slots và tích hợp phân trang.

## 🚀 Tính năng nổi bật

- ⚡ **Virtual Scroll:** Hiển thị danh sách hàng nghìn bản ghi mà không làm giảm hiệu năng trình duyệt.
- 🛠 **Custom Slots:** Cho phép override hiển thị của từng cột một cách dễ dàng.
- 🔗 **Checkbox Selection:** Tích hợp sẵn cột chọn dữ liệu khi có định danh duy nhất.
- 📱 **Flexible Layout:** Hỗ trợ cả chiều rộng cố định (`width`) và chiều rộng co giãn (`minWidth`).
- 📄 **Pagination:** Tích hợp điều hướng phân trang.

---

## 📦 Cấu hình Props (Properties)

| Tên | Kiểu dữ liệu | Mô tả | Mặc định |
| :--- | :--- | :--- | :--- |
| `items` (bắt buộc) | `Array` | Danh sách dữ liệu hiển thị trong bảng. | — |
| `headers` (bắt buộc) | `Array` | Cấu hình các cột (xem chi tiết bên dưới). | `[]` |
| `itemKey` | `String` | Tên trường dùng làm khóa duy nhất cho mỗi dòng. Nếu truyền vào, bảng sẽ hiển thị cột checkbox. | `""` |
| `listSelected` | `Array` | Danh sách các item đang được chọn từ bên ngoài. | — |
| `pagination` | `Object` | Thông tin phân trang: `{TotalRows, TotalPages, PageNumber, PageSize}`. | `{TotalRows: 0, ...}` |
| `customHeight` | `String` | Inline style cho chiều cao vùng scroll (Dùng `calc()` để tối ưu). | `height: calc(100vh - 260px)` |
| `loading` | `Boolean` | Trạng thái đang tải dữ liệu. | `false` |

### Cấu hình chi tiết Headers
Mỗi phần tử trong mảng `headers` có cấu trúc:
- `key` (String - bắt buộc): Tên trường lấy dữ liệu và tên slot tương ứng.
- `title` (String - bắt buộc): Tiêu đề hiển thị ở Header và Footer.
- `width` (Number/String): Chiều rộng cố định (px). Cột sẽ không co giãn.
- `minWidth` (String): Chiều rộng tối thiểu khi dùng flex-grow. Mặc định: `'150px'`.

---

## 🔔 Events

| Tên Event | Dữ liệu trả về | Mô tả |
| :--- | :--- | :--- |
| `@update:listSelected` | `Array` | Trả về mảng các Object item đầy đủ đang được chọn mỗi khi có thay đổi. |
| `@update:pagination` | `Object` | Emit khi người dùng bấm nút "Tiếp tục", trả về object pagination mới với `PageNumber` tăng lên 1. |

---

## 🛠 Slots - Tùy chỉnh hiển thị ô dữ liệu

Mỗi cột có thể được tùy chỉnh bằng cách sử dụng slot với tên là `key` của cột đó. Slot nhận scoped prop `{ item }` là dữ liệu của dòng hiện tại.

```html
<uc-vui-table :items="data" :headers="headers" itemKey="id">
  <template #lead="{ item }">
    <v-text-field
      v-model="item.lead"
      density="compact"
      variant="outlined"
      type="number"
    />
  </template>

  </uc-vui-table>
```

---

## 🕹 Ref Methods

Bạn có thể sử dụng `ref` để gọi các phương thức trực tiếp từ component:

| Method | Mô tả |
| :--- | :--- |
| `scrollToTop()` | Cuộn bảng về vị trí đầu tiên. |
| `scrollToBottom()` | Cuộn bảng xuống vị trí cuối cùng. |

```javascript
const tableRef = ref(null);

// Ví dụ: Cuộn lên đầu sau khi tải dữ liệu mới
const loadData = () => {
  // logic load data...
  tableRef.value.scrollToTop();
}
```

---

## 📝 Ví dụ thực tế

```vue
<template>
  <uc-vui-table
    ref="tableRef"
    :items="dataSource"
    :headers="tableHeaders"
    itemKey="id"
    :listSelected="selectedItems"
    customHeight="height: calc(100vh - 450px)"
    @update:listSelected="onChangeSelected"
  >
    <template #lead="{ item }">
      <v-text-field 
        v-model="item.lead" 
        density="compact"
        variant="outlined" 
        hide-details 
        type="number"
        :disabled="!isRowSelected(item)" 
      />
    </template>

    <template #process="{ item }">
      <v-progress-linear v-model="item.process" color="primary" />
    </template>
  </uc-vui-table>
</template>

<script setup>
const tableHeaders = [
  { key: 'nganh',  title: 'Ngành',   minWidth: '200px' },
  { key: 'lead',   title: 'Lead',    width: 160 },
  { key: 'process',title: 'Process', width: 160 },
];

const dataSource = ref([...]);
const selectedItems = ref([]);

const onChangeSelected = (items) => {
  selectedItems.value = items;
};
</script>
```

---

## ⚠️ Lưu ý quan trọng

1. **itemKey:** Nếu không truyền `itemKey`, cột checkbox sẽ bị ẩn. Đảm bảo `itemKey` là duy nhất (như `id`, `uuid`) để việc lựa chọn dòng không bị lỗi.
2. **Performance:** Khi sử dụng slot để render các component phức tạp (như Input, Select) trong Virtual Scroll, hãy đảm bảo các component đó đã được tối ưu để tránh hiện tượng giật lag khi cuộn nhanh.
3. **listSelected:** Để xóa sạch các lựa chọn từ bên ngoài, chỉ cần gán `listSelected = []`.