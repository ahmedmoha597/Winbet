# Winbet
Betting odds 
<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>WinBet - منصة المراهنات الرياضية</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f2f2f2;
      margin: 0;
      padding: 20px;
      direction: rtl;
    }
    h1 {
      text-align: center;
      color: #333;
    }
    #login-section, #events-section {
      max-width: 500px;
      margin: 20px auto;
      background: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px #ccc;
    }
    label {
      display: block;
      margin-bottom: 8px;
      font-weight: bold;
    }
    input[type="email"], input[type="password"] {
      width: 100%;
      padding: 8px;
      margin-bottom: 12px;
      border: 1px solid #ccc;
      border-radius: 4px;
      box-sizing: border-box;
    }
    button {
      background-color: #28a745;
      border: none;
      color: white;
      padding: 10px 16px;
      cursor: pointer;
      border-radius: 4px;
      font-size: 16px;
    }
    button:hover {
      background-color: #218838;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 12px;
    }
    th, td {
      border: 1px solid #ddd;
      padding: 12px;
      text-align: center;
    }
    th {
      background-color: #28a745;
      color: white;
    }
    .btn-bet {
      background-color: #007bff;
      padding: 6px 10px;
      border-radius: 4px;
      font-size: 14px;
    }
    .btn-bet:hover {
      background-color: #0056b3;
    }
  </style>
</head>
<body>

  <h1>منصة WinBet للمراهنات الرياضية</h1>

  <section id="login-section">
    <h2>تسجيل الدخول</h2>
    <form id="login-form">
      <label for="email">البريد الإلكتروني:</label>
      <input type="email" id="email" required placeholder="example@mail.com" />

      <label for="password">كلمة المرور:</label>
      <input type="password" id="password" required placeholder="********" />

      <button type="submit">تسجيل الدخول</button>
    </form>
  </section>

  <section id="events-section" style="display: none;">
    <h2>الأحداث الرياضية المتاحة</h2>
    <table>
      <thead>
        <tr>
          <th>المباراة</th>
          <th>التاريخ والوقت</th>
          <th>احتمالات الفريق 1</th>
          <th>احتمالات الفريق 2</th>
          <th>الرهان</th>
        </tr>
      </thead>
      <tbody id="events-table-body"></tbody>
    </table>
  </section>

  <script>
    // استدعاء عناصر الصفحة
    const loginSection = document.getElementById('login-section');
    const eventsSection = document.getElementById('events-section');
    const eventsTableBody = document.getElementById('events-table-body');
    const loginForm = document.getElementById('login-form');

    // حفظ البريد الإلكتروني بعد تسجيل الدخول لاستخدامه لاحقاً
    let loggedInEmail = '';

    // دالة لإنشاء صف في جدول الأحداث
    function createEventRow(event) {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${event.match}</td>
        <td>${event.datetime}</td>
        <td>${event.oddsTeam1}</td>
        <td>${event.oddsTeam2}</td>
        <td><button class="btn-bet" onclick="placeBet(${event.id})">راهن الآن</button></td>
      `;
      return tr;
    }

    // عند إرسال نموذج تسجيل الدخول
    loginForm.addEventListener('submit', function(e) {
      e.preventDefault();

      const email = document.getElementById('email').value.trim();
      const password = document.getElementById('password').value.trim();

      fetch('http://localhost:3000/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      })
      .then(res => {
        if (!res.ok) throw new Error('فشل تسجيل الدخول، تحقق من البريد أو كلمة المرور');
        return res.json();
      })
      .then(data => {
        alert(data.message);
        loggedInEmail = email;  // حفظ البريد الإلكتروني

        // جلب الأحداث الرياضية
        return fetch('http://localhost:3000/api/events');
      })
      .then(res => res.json())
      .then(eventsData => {
        loginSection.style.display = 'none';
        eventsSection.style.display = 'block';
        eventsTableBody.innerHTML = '';
        eventsData.forEach(event => {
          eventsTableBody.appendChild(createEventRow(event));
        });
      })
      .catch(err => alert(err.message));
    });

    // دالة إرسال رهان للسيرفر
    function placeBet(eventId) {
      if (!loggedInEmail) {
        alert('يجب تسجيل الدخول أولاً');
        return;
      }

      const amount = prompt("ادخل مبلغ الرهان:");
      if (!amount || isNaN(amount) || amount <= 0) {
        alert("يرجى إدخال مبلغ صالح");
        return;
      }

      const team = prompt("اختر الفريق: 1 أو 2");
      if (team !== '1' && team !== '2') {
        alert("يرجى اختيار الفريق 1 أو 2 فقط");
        return;
      }

      fetch('http://localhost:3000/api/bet', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email: loggedInEmail, eventId, team: Number(team), amount: Number(amount) })
      })
      .then(res => {
        if (!res.ok) throw new Error('حدث خطأ أثناء إرسال الرهان');
        return res.json();
      })
      .then(data => alert(data.message))
      .catch(err => alert(err.message));
    }
  </script>

</body>
</html><!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>WinBet - منصة المراهنات الرياضية</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f2f2f2;
      margin: 0;
      padding: 20px;
      direction: rtl;
    }
    h1 {
      text-align: center;
      color: #333;
    }
    #login-section, #events-section {
      max-width: 500px;
      margin: 20px auto;
      background: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px #ccc;
    }
    label {
      display: block;
      margin-bottom: 8px;
      font-weight: bold;
    }
    input[type="email"], input[type="password"] {
      width: 100%;
      padding: 8px;
      margin-bottom: 12px;
      border: 1px solid #ccc;
      border-radius: 4px;
      box-sizing: border-box;
    }
    button {
      background-color: #28a745;
      border: none;
      color: white;
      padding: 10px 16px;
      cursor: pointer;
      border-radius: 4px;
      font-size: 16px;
    }
    button:hover {
      background-color: #218838;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 12px;
    }
    th, td {
      border: 1px solid #ddd;
      padding: 12px;
      text-align: center;
    }
    th {
      background-color: #28a745;
      color: white;
    }
    .btn-bet {
      background-color: #007bff;
      padding: 6px 10px;
      border-radius: 4px;
      font-size: 14px;
    }
    .btn-bet:hover {
      background-color: #0056b3;
    }
  </style>
</head>
<body>

  <h1>منصة WinBet للمراهنات الرياضية</h1>

  <section id="login-section">
    <h2>تسجيل الدخول</h2>
    <form id="login-form">
      <label for="email">البريد الإلكتروني:</label>
      <input type="email" id="email" required placeholder="example@mail.com" />

      <label for="password">كلمة المرور:</label>
      <input type="password" id="password" required placeholder="********" />

      <button type="submit">تسجيل الدخول</button>
    </form>
  </section>

  <section id="events-section" style="display: none;">
    <h2>الأحداث الرياضية المتاحة</h2>
    <table>
      <thead>
        <tr>
          <th>المباراة</th>
          <th>التاريخ والوقت</th>
          <th>احتمالات الفريق 1</th>
          <th>احتمالات الفريق 2</th>
          <th>الرهان</th>
        </tr>
      </thead>
      <tbody id="events-table-body"></tbody>
    </table>
  </section>

  <script>
    // استدعاء عناصر الصفحة
    const loginSection = document.getElementById('login-section');
    const eventsSection = document.getElementById('events-section');
    const eventsTableBody = document.getElementById('events-table-body');
    const loginForm = document.getElementById('login-form');

    // حفظ البريد الإلكتروني بعد تسجيل الدخول لاستخدامه لاحقاً
    let loggedInEmail = '';

    // دالة لإنشاء صف في جدول الأحداث
    function createEventRow(event) {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${event.match}</td>
        <td>${event.datetime}</td>
        <td>${event.oddsTeam1}</td>
        <td>${event.oddsTeam2}</td>
        <td><button class="btn-bet" onclick="placeBet(${event.id})">راهن الآن</button></td>
      `;
      return tr;
    }

    // عند إرسال نموذج تسجيل الدخول
    loginForm.addEventListener('submit', function(e) {
      e.preventDefault();

      const email = document.getElementById('email').value.trim();
      const password = document.getElementById('password').value.trim();

      fetch('http://localhost:3000/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      })
      .then(res => {
        if (!res.ok) throw new Error('فشل تسجيل الدخول، تحقق من البريد أو كلمة المرور');
        return res.json();
      })
      .then(data => {
        alert(data.message);
        loggedInEmail = email;  // حفظ البريد الإلكتروني

        // جلب الأحداث الرياضية
        return fetch('http://localhost:3000/api/events');
      })
      .then(res => res.json())
      .then(eventsData => {
        loginSection.style.display = 'none';
        eventsSection.style.display = 'block';
        eventsTableBody.innerHTML = '';
        eventsData.forEach(event => {
          eventsTableBody.appendChild(createEventRow(event));
        });
      })
      .catch(err => alert(err.message));
    });

    // دالة إرسال رهان للسيرفر
    function placeBet(eventId) {
      if (!loggedInEmail) {
        alert('يجب تسجيل الدخول أولاً');
        return;
      }

      const amount = prompt("ادخل مبلغ الرهان:");
      if (!amount || isNaN(amount) || amount <= 0) {
        alert("يرجى إدخال مبلغ صالح");
        return;
      }

      const team = prompt("اختر الفريق: 1 أو 2");
      if (team !== '1' && team !== '2') {
        alert("يرجى اختيار الفريق 1 أو 2 فقط");
        return;
      }

      fetch('http://localhost:3000/api/bet', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email: loggedInEmail, eventId, team: Number(team), amount: Number(amount) })
      })
      .then(res => {
        if (!res.ok) throw new Error('حدث خطأ أثناء إرسال الرهان');
        return res.json();
      })
      .then(data => alert(data.message))
      .catch(err => alert(err.message));
    }
  </script>

</body>
</html><!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>WinBet - منصة المراهنات الرياضية</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f2f2f2;
      margin: 0;
      padding: 20px;
      direction: rtl;
    }
    h1 {
      text-align: center;
      color: #333;
    }
    #login-section, #events-section {
      max-width: 500px;
      margin: 20px auto;
      background: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px #ccc;
    }
    label {
      display: block;
      margin-bottom: 8px;
      font-weight: bold;
    }
    input[type="email"], input[type="password"] {
      width: 100%;
      padding: 8px;
      margin-bottom: 12px;
      border: 1px solid #ccc;
      border-radius: 4px;
      box-sizing: border-box;
    }
    button {
      background-color: #28a745;
      border: none;
      color: white;
      padding: 10px 16px;
      cursor: pointer;
      border-radius: 4px;
      font-size: 16px;
    }
    button:hover {
      background-color: #218838;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 12px;
    }
    th, td {
      border: 1px solid #ddd;
      padding: 12px;
      text-align: center;
    }
    th {
      background-color: #28a745;
      color: white;
    }
    .btn-bet {
      background-color: #007bff;
      padding: 6px 10px;
      border-radius: 4px;
      font-size: 14px;
    }
    .btn-bet:hover {
      background-color: #0056b3;
    }
  </style>
</head>
<body>

  <h1>منصة WinBet للمراهنات الرياضية</h1>

  <section id="login-section">
    <h2>تسجيل الدخول</h2>
    <form id="login-form">
      <label for="email">البريد الإلكتروني:</label>
      <input type="email" id="email" required placeholder="example@mail.com" />

      <label for="password">كلمة المرور:</label>
      <input type="password" id="password" required placeholder="********" />

      <button type="submit">تسجيل الدخول</button>
    </form>
  </section>

  <section id="events-section" style="display: none;">
    <h2>الأحداث الرياضية المتاحة</h2>
    <table>
      <thead>
        <tr>
          <th>المباراة</th>
          <th>التاريخ والوقت</th>
          <th>احتمالات الفريق 1</th>
          <th>احتمالات الفريق 2</th>
          <th>الرهان</th>
        </tr>
      </thead>
      <tbody id="events-table-body"></tbody>
    </table>
  </section>

  <script>
    // استدعاء عناصر الصفحة
    const loginSection = document.getElementById('login-section');
    const eventsSection = document.getElementById('events-section');
    const eventsTableBody = document.getElementById('events-table-body');
    const loginForm = document.getElementById('login-form');

    // حفظ البريد الإلكتروني بعد تسجيل الدخول لاستخدامه لاحقاً
    let loggedInEmail = '';

    // دالة لإنشاء صف في جدول الأحداث
    function createEventRow(event) {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${event.match}</td>
        <td>${event.datetime}</td>
        <td>${event.oddsTeam1}</td>
        <td>${event.oddsTeam2}</td>
        <td><button class="btn-bet" onclick="placeBet(${event.id})">راهن الآن</button></td>
      `;
      return tr;
    }

    // عند إرسال نموذج تسجيل الدخول
    loginForm.addEventListener('submit', function(e) {
      e.preventDefault();

      const email = document.getElementById('email').value.trim();
      const password = document.getElementById('password').value.trim();

      fetch('http://localhost:3000/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      })
      .then(res => {
        if (!res.ok) throw new Error('فشل تسجيل الدخول، تحقق من البريد أو كلمة المرور');
        return res.json();
      })
      .then(data => {
        alert(data.message);
        loggedInEmail = email;  // حفظ البريد الإلكتروني

        // جلب الأحداث الرياضية
        return fetch('http://localhost:3000/api/events');
      })
      .then(res => res.json())
      .then(eventsData => {
        loginSection.style.display = 'none';
        eventsSection.style.display = 'block';
        eventsTableBody.innerHTML = '';
        eventsData.forEach(event => {
          eventsTableBody.appendChild(createEventRow(event));
        });
      })
      .catch(err => alert(err.message));
    });

    // دالة إرسال رهان للسيرفر
    function placeBet(eventId) {
      if (!loggedInEmail) {
        alert('يجب تسجيل الدخول أولاً');
        return;
      }

      const amount = prompt("ادخل مبلغ الرهان:");
      if (!amount || isNaN(amount) || amount <= 0) {
        alert("يرجى إدخال مبلغ صالح");
        return;
      }

      const team = prompt("اختر الفريق: 1 أو 2");
      if (team !== '1' && team !== '2') {
        alert("يرجى اختيار الفريق 1 أو 2 فقط");
        return;
      }

      fetch('http://localhost:3000/api/bet', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email: loggedInEmail, eventId, team: Number(team), amount: Number(amount) })
      })
      .then(res => {
        if (!res.ok) throw new Error('حدث خطأ أثناء إرسال الرهان');
        return res.json();
      })
      .then(data => alert(data.message))
      .catch(err => alert(err.message));
    }
  </script>

</body>
</html><!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>WinBet - منصة المراهنات الرياضية</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f2f2f2;
      margin: 0;
      padding: 20px;
      direction: rtl;
    }
    h1 {
      text-align: center;
      color: #333;
    }
    #login-section, #events-section {
      max-width: 500px;
      margin: 20px auto;
      background: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px #ccc;
    }
    label {
      display: block;
      margin-bottom: 8px;
      font-weight: bold;
    }
    input[type="email"], input[type="password"] {
      width: 100%;
      padding: 8px;
      margin-bottom: 12px;
      border: 1px solid #ccc;
      border-radius: 4px;
      box-sizing: border-box;
    }
    button {
      background-color: #28a745;
      border: none;
      color: white;
      padding: 10px 16px;
      cursor: pointer;
      border-radius: 4px;
      font-size: 16px;
    }
    button:hover {
      background-color: #218838;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 12px;
    }
    th, td {
      border: 1px solid #ddd;
      padding: 12px;
      text-align: center;
    }
    th {
      background-color: #28a745;
      color: white;
    }
    .btn-bet {
      background-color: #007bff;
      padding: 6px 10px;
      border-radius: 4px;
      font-size: 14px;
    }
    .btn-bet:hover {
      background-color: #0056b3;
    }
  </style>
</head>
<body>

  <h1>منصة WinBet للمراهنات الرياضية</h1>

  <section id="login-section">
    <h2>تسجيل الدخول</h2>
    <form id="login-form">
      <label for="email">البريد الإلكتروني:</label>
      <input type="email" id="email" required placeholder="example@mail.com" />

      <label for="password">كلمة المرور:</label>
      <input type="password" id="password" required placeholder="********" />

      <button type="submit">تسجيل الدخول</button>
    </form>
  </section>

  <section id="events-section" style="display: none;">
    <h2>الأحداث الرياضية المتاحة</h2>
    <table>
      <thead>
        <tr>
          <th>المباراة</th>
          <th>التاريخ والوقت</th>
          <th>احتمالات الفريق 1</th>
          <th>احتمالات الفريق 2</th>
          <th>الرهان</th>
        </tr>
      </thead>
      <tbody id="events-table-body"></tbody>
    </table>
  </section>

  <script>
    // استدعاء عناصر الصفحة
    const loginSection = document.getElementById('login-section');
    const eventsSection = document.getElementById('events-section');
    const eventsTableBody = document.getElementById('events-table-body');
    const loginForm = document.getElementById('login-form');

    // حفظ البريد الإلكتروني بعد تسجيل الدخول لاستخدامه لاحقاً
    let loggedInEmail = '';

    // دالة لإنشاء صف في جدول الأحداث
    function createEventRow(event) {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${event.match}</td>
        <td>${event.datetime}</td>
        <td>${event.oddsTeam1}</td>
        <td>${event.oddsTeam2}</td>
        <td><button class="btn-bet" onclick="placeBet(${event.id})">راهن الآن</button></td>
      `;
      return tr;
    }

    // عند إرسال نموذج تسجيل الدخول
    loginForm.addEventListener('submit', function(e) {
      e.preventDefault();

      const email = document.getElementById('email').value.trim();
      const password = document.getElementById('password').value.trim();

      fetch('http://localhost:3000/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      })
      .then(res => {
        if (!res.ok) throw new Error('فشل تسجيل الدخول، تحقق من البريد أو كلمة المرور');
        return res.json();
      })
      .then(data => {
        alert(data.message);
        loggedInEmail = email;  // حفظ البريد الإلكتروني

        // جلب الأحداث الرياضية
        return fetch('http://localhost:3000/api/events');
      })
      .then(res => res.json())
      .then(eventsData => {
        loginSection.style.display = 'none';
        eventsSection.style.display = 'block';
        eventsTableBody.innerHTML = '';
        eventsData.forEach(event => {
          eventsTableBody.appendChild(createEventRow(event));
        });
      })
      .catch(err => alert(err.message));
    });

    // دالة إرسال رهان للسيرفر
    function placeBet(eventId) {
      if (!loggedInEmail) {
        alert('يجب تسجيل الدخول أولاً');
        return;
      }

      const amount = prompt("ادخل مبلغ الرهان:");
      if (!amount || isNaN(amount) || amount <= 0) {
        alert("يرجى إدخال مبلغ صالح");
        return;
      }

      const team = prompt("اختر الفريق: 1 أو 2");
      if (team !== '1' && team !== '2') {
        alert("يرجى اختيار الفريق 1 أو 2 فقط");
        return;
      }

      fetch('http://localhost:3000/api/bet', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email: loggedInEmail, eventId, team: Number(team), amount: Number(amount) })
      })
      .then(res => {
        if (!res.ok) throw new Error('حدث خطأ أثناء إرسال الرهان');
        return res.json();
      })
      .then(data => alert(data.message))
      .catch(err => alert(err.message));
    }
  </script>

</body>
</html><!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>WinBet - منصة المراهنات الرياضية</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f2f2f2;
      margin: 0;
      padding: 20px;
      direction: rtl;
    }
    h1 {
      text-align: center;
      color: #333;
    }
    #login-section, #events-section {
      max-width: 500px;
      margin: 20px auto;
      background: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px #ccc;
    }
    label {
      display: block;
      margin-bottom: 8px;
      font-weight: bold;
    }
    input[type="email"], input[type="password"] {
      width: 100%;
      padding: 8px;
      margin-bottom: 12px;
      border: 1px solid #ccc;
      border-radius: 4px;
      box-sizing: border-box;
    }
    button {
      background-color: #28a745;
      border: none;
      color: white;
      padding: 10px 16px;
      cursor: pointer;
      border-radius: 4px;
      font-size: 16px;
    }
    button:hover {
      background-color: #218838;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 12px;
    }
    th, td {
      border: 1px solid #ddd;
      padding: 12px;
      text-align: center;
    }
    th {
      background-color: #28a745;
      color: white;
    }
    .btn-bet {
      background-color: #007bff;
      padding: 6px 10px;
      border-radius: 4px;
      font-size: 14px;
    }
    .btn-bet:hover {
      background-color: #0056b3;
    }
  </style>
</head>
<body>

  <h1>منصة WinBet للمراهنات الرياضية</h1>

  <section id="login-section">
    <h2>تسجيل الدخول</h2>
    <form id="login-form">
      <label for="email">البريد الإلكتروني:</label>
      <input type="email" id="email" required placeholder="example@mail.com" />

      <label for="password">كلمة المرور:</label>
      <input type="password" id="password" required placeholder="********" />

      <button type="submit">تسجيل الدخول</button>
    </form>
  </section>

  <section id="events-section" style="display: none;">
    <h2>الأحداث الرياضية المتاحة</h2>
    <table>
      <thead>
        <tr>
          <th>المباراة</th>
          <th>التاريخ والوقت</th>
          <th>احتمالات الفريق 1</th>
          <th>احتمالات الفريق 2</th>
          <th>الرهان</th>
        </tr>
      </thead>
      <tbody id="events-table-body"></tbody>
    </table>
  </section>

  <script>
    // استدعاء عناصر الصفحة
    const loginSection = document.getElementById('login-section');
    const eventsSection = document.getElementById('events-section');
    const eventsTableBody = document.getElementById('events-table-body');
    const loginForm = document.getElementById('login-form');

    // حفظ البريد الإلكتروني بعد تسجيل الدخول لاستخدامه لاحقاً
    let loggedInEmail = '';

    // دالة لإنشاء صف في جدول الأحداث
    function createEventRow(event) {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${event.match}</td>
        <td>${event.datetime}</td>
        <td>${event.oddsTeam1}</td>
        <td>${event.oddsTeam2}</td>
        <td><button class="btn-bet" onclick="placeBet(${event.id})">راهن الآن</button></td>
      `;
      return tr;
    }

    // عند إرسال نموذج تسجيل الدخول
    loginForm.addEventListener('submit', function(e) {
      e.preventDefault();

      const email = document.getElementById('email').value.trim();
      const password = document.getElementById('password').value.trim();

      fetch('http://localhost:3000/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      })
      .then(res => {
        if (!res.ok) throw new Error('فشل تسجيل الدخول، تحقق من البريد أو كلمة المرور');
        return res.json();
      })
      .then(data => {
        alert(data.message);
        loggedInEmail = email;  // حفظ البريد الإلكتروني

        // جلب الأحداث الرياضية
        return fetch('http://localhost:3000/api/events');
      })
      .then(res => res.json())
      .then(eventsData => {
        loginSection.style.display = 'none';
        eventsSection.style.display = 'block';
        eventsTableBody.innerHTML = '';
        eventsData.forEach(event => {
          eventsTableBody.appendChild(createEventRow(event));
        });
      })
      .catch(err => alert(err.message));
    });

    // دالة إرسال رهان للسيرفر
    function placeBet(eventId) {
      if (!loggedInEmail) {
        alert('يجب تسجيل الدخول أولاً');
        return;
      }

      const amount = prompt("ادخل مبلغ الرهان:");
      if (!amount || isNaN(amount) || amount <= 0) {
        alert("يرجى إدخال مبلغ صالح");
        return;
      }

      const team = prompt("اختر الفريق: 1 أو 2");
      if (team !== '1' && team !== '2') {
        alert("يرجى اختيار الفريق 1 أو 2 فقط");
        return;
      }

      fetch('http://localhost:3000/api/bet', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email: loggedInEmail, eventId, team: Number(team), amount: Number(amount) })
      })
      .then(res => {
        if (!res.ok) throw new Error('حدث خطأ أثناء إرسال الرهان');
        return res.json();
      })
      .then(data => alert(data.message))
      .catch(err => alert(err.message));
    }
  </script>

</body>
</html>
