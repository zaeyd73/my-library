# my-library
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>مكتبتي الشخصية</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f4f6f9;
            color: #333;
            margin: 0;
            padding: 20px;
        }
        .container {
            max-width: 900px;
            margin: auto;
            background: #fff;
            padding: 25px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.05);
        }
        h1 {
            text-align: center;
            color: #2c3e50;
            margin-bottom: 25px;
        }
        .form-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 15px;
            margin-bottom: 20px;
            background: #f8f9fa;
            padding: 15px;
            border-radius: 8px;
        }
        input, select {
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 6px;
            font-size: 14px;
        }
        button {
            background-color: #27ae60;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 15px;
            font-weight: bold;
            transition: background 0.2s;
        }
        button:hover {
            background-color: #219653;
        }
        .search-box input {
            width: 100%;
            padding: 12px;
            font-size: 16px;
            border: 2px solid #3498db;
            border-radius: 6px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 10px;
        }
        th, td {
            padding: 12px;
            text-align: right;
            border-bottom: 1px solid #ddd;
        }
        th {
            background-color: #34495e;
            color: white;
        }
        tr:hover {
            background-color: #f1f1f1;
        }
        .delete-btn {
            background-color: #e74c3c;
            padding: 6px 12px;
            font-size: 13px;
        }
        .edit-btn {
            background-color: #f1c40f;
            padding: 6px 12px;
            font-size: 13px;
            margin-left: 5px;
        }
        .empty-msg {
            text-align: center;
            color: #7f8c8d;
            padding: 20px;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>📚 نظام إدارة وفهرسة المكتبة الشخصية</h1>

    <!-- نموذج إضافة كتاب -->
    <div class="form-grid">
        <input type="text" id="bookTitle" placeholder="عنوان الكتاب" required>
        <input type="text" id="bookAuthor" placeholder="المؤلف">
        <input type="text" id="bookLocation" placeholder="مكانه (مثلاً: الرف الثاني)">
        
        <select id="bookCategory">
            <option value="">التصنيف</option>
            <option>رواية</option>
            <option>دين</option>
            <option>تاريخ</option>
            <option>علوم</option>
            <option>تطوير ذات</option>
            <option>أدب</option>
        </select>

        <button onclick="saveBook()">حفظ</button>
    </div>

    <!-- شريط البحث -->
    <div class="search-box">
        <input type="text" id="searchInput" placeholder="🔍 ابحث عن كتاب..." onkeyup="filterBooks()">
    </div>

    <!-- جدول عرض الكتب -->
    <table>
        <thead>
            <tr>
                <th>العنوان</th>
                <th>المؤلف</th>
                <th>التصنيف</th>
                <th>المكان</th>
                <th>الإجراء</th>
            </tr>
        </thead>
        <tbody id="bookTableBody"></tbody>
    </table>
</div>

<script>
    let books = JSON.parse(localStorage.getItem('myLibraryBooks')) || [];
    let editIndex = null;

    function renderBooks(list = books) {
        const tbody = document.getElementById('bookTableBody');
        tbody.innerHTML = '';

        if (list.length === 0) {
            tbody.innerHTML = `<tr><td colspan="5" class="empty-msg">لا توجد كتب مسجلة.</td></tr>`;
            return;
        }

        list.forEach((book, index) => {
            tbody.innerHTML += `
                <tr>
                    <td><strong>${book.title}</strong></td>
                    <td>${book.author || 'غير مذكور'}</td>
                    <td>${book.category || 'غير محدد'}</td>
                    <td>📍 ${book.location || 'غير محدد'}</td>
                    <td>
                        <button class="edit-btn" onclick="editBook(${index})">تعديل</button>
                        <button class="delete-btn" onclick="deleteBook(${index})">حذف</button>
                    </td>
                </tr>
            `;
        });
    }

    function saveBook() {
        const title = document.getElementById('bookTitle').value.trim();
        const author = document.getElementById('bookAuthor').value.trim();
        const location = document.getElementById('bookLocation').value.trim();
        const category = document.getElementById('bookCategory').value;

        if (!title) {
            alert('يجب إدخال عنوان الكتاب.');
            return;
        }

        const bookData = { title, author, location, category };

        if (editIndex !== null) {
            books[editIndex] = bookData;
            editIndex = null;
        } else {
            books.push(bookData);
        }

        localStorage.setItem('myLibraryBooks', JSON.stringify(books));
        clearForm();
        renderBooks();
    }

    function editBook(index) {
        const book = books[index];
        document.getElementById('bookTitle').value = book.title;
        document.getElementById('bookAuthor').value = book.author;
        document.getElementById('bookLocation').value = book.location;
        document.getElementById('bookCategory').value = book.category;

        editIndex = index;
    }

    function deleteBook(index) {
        if (confirm('هل تريد حذف هذا الكتاب؟')) {
            books.splice(index, 1);
            localStorage.setItem('myLibraryBooks', JSON.stringify(books));
            renderBooks();
        }
    }

    function filterBooks() {
        const q = document.getElementById('searchInput').value.toLowerCase();
        const filtered = books.filter(b =>
            b.title.toLowerCase().includes(q) ||
            (b.author || '').toLowerCase().includes(q) ||
            (b.location || '').toLowerCase().includes(q) ||
            (b.category || '').toLowerCase().includes(q)
        );
        renderBooks(filtered);
    }

    function clearForm() {
        document.getElementById('bookTitle').value = '';
        document.getElementById('bookAuthor').value = '';
        document.getElementById('bookLocation').value = '';
        document.getElementById('bookCategory').value = '';
    }

    renderBooks();
</script>

</body>
</html>
