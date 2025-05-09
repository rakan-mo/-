# -import React, { useState, useEffect } from 'react';

const App = () => {
  const [activeTab, setActiveTab] = useState('dashboard');
  const [user, setUser] = useState(null);
  const [accidents, setAccidents] = useState([]);
  const [employees, setEmployees] = useState([
    { id: 1, name: "أحمد محمد", role: "مساعد تحقيق", reports: 7 },
    { id: 2, name: "سعود علي", role: "ضابط أمن", reports: 12 },
    { id: 3, name: "فهد خالد", role: "مهندس سلامة", reports: 9 },
  ]);

  const [newAccident, setNewAccident] = useState({
    location: "",
    date: "",
    time: "",
    description: "",
    employeeIds: [],
    images: [],
  });

  const [notification, setNotification] = useState("");

  // Load data from localStorage on component mount
  useEffect(() => {
    const savedAccidents = JSON.parse(localStorage.getItem("accidents")) || [];
    const savedUser = JSON.parse(localStorage.getItem("user"));
    setAccidents(savedAccidents);
    setUser(savedUser);
  }, []);

  // Save accidents to localStorage whenever it changes
  useEffect(() => {
    localStorage.setItem("accidents", JSON.stringify(accidents));
  }, [accidents]);

  // Login handler
  const handleLogin = (e) => {
    e.preventDefault();
    const username = e.target.username.value;
    const password = e.target.password.value;

    if ((username === "admin" && password === "admin") || 
        (username === "victim" && password === "victim")) {
      const userData = { username, role: username === "admin" ? "admin" : "victim" };
      setUser(userData);
      localStorage.setItem("user", JSON.stringify(userData));
    } else {
      showNotification("اسم المستخدم أو كلمة المرور غير صحيحة");
    }
  };

  // Logout handler
  const handleLogout = () => {
    setUser(null);
    localStorage.removeItem("user");
  };

  // Add new accident
  const handleAddAccident = (e) => {
    e.preventDefault();

    if (!newAccident.location || !newAccident.date || !newAccident.time || !newAccident.description) {
      showNotification("يرجى ملء جميع الحقول المطلوبة.");
      return;
    }

    const newId = accidents.length > 0 ? Math.max(...accidents.map(a => a.id)) + 1 : 1;
    const selectedEmployees = employees.filter(emp => newAccident.employeeIds.includes(emp.id));
    const employeeNames = selectedEmployees.map(emp => emp.name);

    const accidentToAdd = {
      ...newAccident,
      id: newId,
      employees: employeeNames,
    };

    setAccidents([accidentToAdd, ...accidents]);
    setNewAccident({ location: "", date: "", time: "", description: "", employeeIds: [], images: [] });
    setActiveTab('dashboard');
    showNotification("تمت إضافة الحادث بنجاح!");
  };

  // Handle image upload
  const handleImageUpload = (e) => {
    const files = Array.from(e.target.files);
    const imageUrls = files.map(file => URL.createObjectURL(file));
    setNewAccident(prev => ({
      ...prev,
      images: [...prev.images, ...imageUrls]
    }));
  };

  // Remove image
  const removeImage = (index) => {
    const updatedImages = [...newAccident.images];
    updatedImages.splice(index, 1);
    setNewAccident({ ...newAccident, images: updatedImages });
  };

  // Show notification
  const showNotification = (message) => {
    setNotification(message);
    setTimeout(() => setNotification(""), 3000);
  };

  // Print report
  const printReport = (accident) => {
    const win = window.open('', '', 'height=600,width=800');
    win.document.write(`
      <html>
        <head><title>تقرير الحادث</title></head>
        <body style="font-family:sans-serif; padding:20px;">
          <h1 style="color:#4F46E5;">تقرير الحادث #${accident.id}</h1>
          <p><strong>الموقع:</strong> ${accident.location}</p>
          <p><strong>التاريخ:</strong> ${accident.date}</p>
          <p><strong>الوقت:</strong> ${accident.time}</p>
          <p><strong>الوصف:</strong> ${accident.description}</p>
          <p><strong>المشاركين:</strong> ${accident.employees.join(', ')}</p>
          <hr/>
          <p style="color:#6B7280;">تم إنشاء التقرير بواسطة نظام إدارة الحوادث.</p>
        </body>
      </html>
    `);
    win.document.close();
    win.print();
  };

  return (
    <div className="min-h-screen bg-gray-100">
      {/* Notification */}
      {notification && (
        <div className="fixed top-4 right-4 bg-green-500 text-white px-6 py-3 rounded shadow-lg z-50 animate-bounce">
          {notification}
        </div>
      )}

      {/* Header */}
      <header className="bg-indigo-600 text-white shadow-md">
        <div className="max-w-7xl mx-auto px-6 py-4 flex justify-between items-center">
          <h1 className="text-2xl font-bold">نظام إدارة الحوادث</h1>
          {user ? (
            <div className="flex items-center space-x-4">
              <span className="hidden md:inline">مرحباً، {user.username} ({user.role})</span>
              <button onClick={handleLogout} className="px-4 py-2 bg-red-500 rounded hover:bg-red-600 transition">تسجيل خروج</button>
            </div>
          ) : (
            <button onClick={() => setActiveTab('login')} className="px-4 py-2 bg-white text-indigo-600 rounded hover:bg-indigo-100 transition">تسجيل دخول</button>
          )}
        </div>
      </header>

      {/* Login Page */}
      {!user && activeTab === 'login' && (
        <div className="max-w-md mx-auto mt-20 p-8 bg-white rounded-lg shadow-lg">
          <h2 className="text-2xl font-bold mb-6 text-center">تسجيل الدخول</h2>
          <form onSubmit={handleLogin}>
            <div className="mb-4">
              <label className="block text-sm font-medium text-gray-700 mb-1">اسم المستخدم</label>
              <input type="text" name="username" required className="w-full border border-gray-300 rounded px-3 py-2 focus:outline-none focus:ring-2 focus:ring-indigo-500" />
            </div>
            <div className="mb-6">
              <label className="block text-sm font-medium text-gray-700 mb-1">كلمة المرور</label>
              <input type="password" name="password" required className="w-full border border-gray-300 rounded px-3 py-2 focus:outline-none focus:ring-2 focus:ring-indigo-500" />
            </div>
            <button type="submit" className="w-full bg-indigo-600 text-white py-2 rounded hover:bg-indigo-700 transition">دخول</button>
          </form>
        </div>
      )}

      {/* Dashboard & Tabs */}
      {user && (
        <div className="max-w-7xl mx-auto px-6 py-8">
          <nav className="flex space-x-4 mb-8 overflow-x-auto pb-2">
            <button onClick={() => setActiveTab('dashboard')} className={`px-4 py-2 rounded ${activeTab === 'dashboard' ? 'bg-indigo-600 text-white' : 'bg-gray-200'}`}>لوحة التحكم</button>
            <button onClick={() => setActiveTab('add-accident')} className={`px-4 py-2 rounded ${activeTab === 'add-accident' ? 'bg-indigo-600 text-white' : 'bg-gray-200'}`}>إضافة حادث</button>
            <button onClick={() => setActiveTab('employees')} className={`px-4 py-2 rounded ${activeTab === 'employees' ? 'bg-indigo-600 text-white' : 'bg-gray-200'}`}>إدارة الموظفين</button>
            <button onClick={() => setActiveTab('reports')} className={`px-4 py-2 rounded ${activeTab === 'reports' ? 'bg-indigo-600 text-white' : 'bg-gray-200'}`}>تقارير الموظفين</button>
          </nav>

          {/* Dashboard */}
          {activeTab === 'dashboard' && (
            <div>
              <h2 className="text-2xl font-bold mb-6">آخر الحوادث</h2>
              <div className="space-y-6">
                {accidents.length > 0 ? (
                  accidents.map(acc => (
                    <div key={acc.id} className="bg-white p-6 rounded-lg shadow">
                      <div className="flex flex-col md:flex-row justify-between">
                        <div>
                          <h3 className="text-xl font-semibold">{acc.location}</h3>
                          <p className="text-gray-600">{acc.date} | {acc.time}</p>
                          <p className="mt-2">{acc.description}</p>
                          <div className="mt-3">
                            <span className="text-sm font-medium text-gray-500">المشاركون:</span>
                            <ul className="list-disc list-inside text-gray-600">
                              {acc.employees.map((emp, idx) => (
                                <li key={idx}>{emp}</li>
                              ))}
                            </ul>
                          </div>
                        </div>
                        <div className="mt-4 md:mt-0">
                          {acc.images.length > 0 && (
                            <img src={acc.images[0]} alt="حادث" className="rounded w-40 h-24 object-cover" />
                          )}
                          <button onClick={() => printReport(acc)} className="mt-2 text-indigo-600 underline">طباعة التقرير</button>
                        </div>
                      </div>
                    </div>
                  ))
                ) : (
                  <p className="text-center text-gray-500">لا توجد حوادث بعد.</p>
                )}
              </div>
            </div>
          )}

          {/* Add Accident Form */}
          {activeTab === 'add-accident' && (
            <div className="bg-white p-6 rounded-lg shadow">
              <h2 className="text-2xl font-bold mb-6">إضافة حادث جديد</h2>
              <form onSubmit={handleAddAccident} className="space-y-4">
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">موقع الحادث</label>
                  <input type="text" value={newAccident.location} onChange={(e) => setNewAccident({...newAccident, location: e.target.value})} required className="w-full border border-gray-300 rounded px-3 py-2" />
                </div>
                <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">التاريخ</label>
                    <input type="date" value={newAccident.date} onChange={(e) => setNewAccident({...newAccident, date: e.target.value})} required className="w-full border border-gray-300 rounded px-3 py-2" />
                  </div>
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">الوقت</label>
                    <input type="time" value={newAccident.time} onChange={(e) => setNewAccident({...newAccident, time: e.target.value})} required className="w-full border border-gray-300 rounded px-3 py-2" />
                  </div>
                </div>
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">وصف الحادث</label>
                  <textarea rows="3" value={newAccident.description} onChange={(e) => setNewAccident({...newAccident, description: e.target.value})} required className="w-full border border-gray-300 rounded px-3 py-2"></textarea>
                </div>
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">اختيار الموظفين</label>
                  <select multiple value={newAccident.employeeIds} onChange={(e) => setNewAccident({...newAccident, employeeIds: Array.from(e.target.selectedOptions, option => parseInt(option.value))})} className="w-full border border-gray-300 rounded px-3 py-2">
                    {employees.map(emp => (
                      <option key={emp.id} value={emp.id}>{emp.name} - {emp.role}</option>
                    ))}
                  </select>
                </div>
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">تحميل صور الحادث</label>
                  <input type="file" accept="image/*" multiple onChange={handleImageUpload} className="w-full border border-gray-300 rounded px-3 py-2" />
                  <div className="mt-2 flex flex-wrap gap-2">
                    {newAccident.images.map((img, index) => (
                      <div key={index} className="relative">
                        <img src={img} alt="مُرفق" className="w-20 h-20 object-cover rounded" />
                        <button type="button" onClick={() => removeImage(index)} className="absolute -top-2 -right-2 bg-red-500 text-white rounded-full w-5 h-5 flex items-center justify-center">x</button>
                      </div>
                    ))}
                  </div>
                </div>
                <div className="flex space-x-4">
                  <button type="submit" className="px-6 py-2 bg-indigo-600 text-white rounded hover:bg-indigo-700">حفظ الحادث</button>
                  <button type="button" onClick={() => setActiveTab('dashboard')} className="px-6 py-2 bg-gray-300 rounded hover:bg-gray-400">إلغاء</button>
                </div>
              </form>
            </div>
          )}

          {/* Employees List */}
          {activeTab === 'employees' && (
            <div className="bg-white p-6 rounded-lg shadow">
              <h2 className="text-2xl font-bold mb-6">قائمة الموظفين</h2>
              <table className="w-full table-auto">
                <thead>
                  <tr className="bg-gray-100">
                    <th className="px-4 py-2 text-left">الاسم</th>
                    <th className="px-4 py-2 text-left">الوظيفة</th>
                    <th className="px-4 py-2 text-left">عدد التقارير</th>
                  </tr>
                </thead>
                <tbody>
                  {employees.map(emp => (
                    <tr key={emp.id} className="border-t">
                      <td className="px-4 py-3">{emp.name}</td>
                      <td className="px-4 py-3">{emp.role}</td>
                      <td className="px-4 py-3">{emp.reports}</td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          )}

          {/* Employee Reports */}
          {activeTab === 'reports' && (
            <div className="bg-white p-6 rounded-lg shadow">
              <h2 className="text-2xl font-bold mb-6">تقارير الموظفين</h2>
              <div className="space-y-6">
                {employees.map(emp => (
                  <div key={emp.id} className="border-b pb-4">
                    <h3 className="text-lg font-semibold">{emp.name} - {emp.role}</h3>
                    <p>عدد التقارير: {emp.reports}</p>
                    <button onClick={() => {
                      const reportContent = `
                        <html>
                          <head><title>تقرير موظف</title></head>
                          <body style="font-family:sans-serif; padding:20px;">
                            <h1 style="color:#4F46E5;">تقرير موظف: ${emp.name}</h1>
                            <p><strong>الوظيفة:</strong> ${emp.role}</p>
                            <p><strong>عدد التقارير:</strong> ${emp.reports}</p>
                            <hr/>
                            <p style="color:#6B7280;">تم إنشاء التقرير بواسطة نظام إدارة الحوادث.</p>
                          </body>
                        </html>
                      `;
                      const win = window.open('', '', 'height=600,width=800');
                      win.document.write(reportContent);
                      win.document.close();
                      win.print();
                    }} className="text-indigo-600 underline mt-2 inline-block">طباعة التقرير</button>
                  </div>
                ))}
              </div>
            </div>
          )}
        </div>
      )}
    </div>
  );
};

export default App;ص
