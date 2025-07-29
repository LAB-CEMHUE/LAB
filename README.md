<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CEM HUẾ - Hệ thống Quản lý Phòng thí nghiệm</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <script src="https://apis.google.com/js/api.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/qrcode@1.5.3/build/qrcode.min.js"></script>
    <style>
        .fade-in { animation: fadeIn 0.3s ease-in; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        .hover-scale { transition: transform 0.2s; }
        .hover-scale:hover { transform: scale(1.02); }
        .file-preview { max-height: 500px; overflow-y: auto; }
        .sample-connection { border-left: 3px solid #3b82f6; background: linear-gradient(90deg, #eff6ff 0%, transparent 100%); }
        .alert-pulse { animation: pulse 2s infinite; }
        @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.7; } }
        .drag-drop-zone { 
            border: 2px dashed #d1d5db; 
            transition: all 0.3s ease;
        }
        .drag-drop-zone.dragover { 
            border-color: #3b82f6; 
            background-color: #eff6ff; 
        }
        .progress-bar {
            transition: width 0.3s ease;
        }
        .file-type-icon {
            width: 40px;
            height: 40px;
            display: flex;
            align-items: center;
            justify-content: center;
            border-radius: 8px;
        }
        .word-icon { background: #2563eb; color: white; }
        .excel-icon { background: #16a34a; color: white; }
        .pdf-icon { background: #dc2626; color: white; }
        .image-icon { background: #7c3aed; color: white; }
        .preview-container {
            max-height: 70vh;
            overflow-y: auto;
        }
        .notification {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 1000;
            padding: 12px 20px;
            border-radius: 8px;
            color: white;
            font-weight: 500;
            transform: translateX(400px);
            transition: transform 0.3s ease;
        }
        .notification.show {
            transform: translateX(0);
        }
        .notification.success { background: #10b981; }
        .notification.error { background: #ef4444; }
        .notification.warning { background: #f59e0b; }
        .notification.info { background: #3b82f6; }
    </style>
</head>
<body class="bg-gradient-to-br from-green-50 to-emerald-100 min-h-screen relative">
    <!-- Environmental Background Pattern -->
    <div class="fixed inset-0 opacity-5 pointer-events-none" style="background-image: url('data:image/svg+xml,<svg xmlns=&quot;http://www.w3.org/2000/svg&quot; viewBox=&quot;0 0 100 100&quot;><defs><pattern id=&quot;env-pattern&quot; x=&quot;0&quot; y=&quot;0&quot; width=&quot;20&quot; height=&quot;20&quot; patternUnits=&quot;userSpaceOnUse&quot;><circle cx=&quot;10&quot; cy=&quot;10&quot; r=&quot;2&quot; fill=&quot;%23059669&quot; opacity=&quot;0.3&quot;/><path d=&quot;M5,15 Q10,10 15,15&quot; stroke=&quot;%23059669&quot; stroke-width=&quot;0.5&quot; fill=&quot;none&quot; opacity=&quot;0.2&quot;/><rect x=&quot;2&quot; y=&quot;2&quot; width=&quot;3&quot; height=&quot;6&quot; fill=&quot;%23059669&quot; opacity=&quot;0.1&quot;/></pattern></defs><rect width=&quot;100&quot; height=&quot;100&quot; fill=&quot;url(%23env-pattern)&quot;/></svg>'); background-size: 200px 200px;"></div>
    
    <!-- Header -->
    <header class="bg-white shadow-lg border-b-4 border-green-600 relative z-10">
        <div class="container mx-auto px-4 lg:px-6 py-4">
            <div class="flex justify-between items-center">
                <div class="flex items-center space-x-3">
                    <div class="w-10 h-10 lg:w-12 lg:h-12 bg-green-600 rounded-xl flex items-center justify-center">
                        <i class="fas fa-flask text-white text-lg lg:text-xl"></i>
                    </div>
                    <div>
                        <h1 class="text-xl lg:text-2xl font-bold text-gray-800">CEM HUẾ</h1>
                        <p class="text-xs lg:text-sm text-gray-600 hidden sm:block">Hệ thống Quản lý Phòng thí nghiệm</p>
                    </div>
                </div>
                <div class="flex items-center space-x-2 lg:space-x-4">
                    <!-- Daily Stats -->
                    <div class="text-center">
                        <div class="text-lg font-bold text-green-600" id="dailySamples">0</div>
                        <div class="text-xs text-gray-500 hidden sm:block">Mẫu nhận hôm nay</div>
                    </div>
                    
                    <!-- Backup Controls -->
                    <div class="flex items-center space-x-1">
                        <button onclick="exportAllData()" class="bg-blue-600 hover:bg-blue-700 text-white px-2 py-1 rounded text-xs" title="Backup dữ liệu">
                            <i class="fas fa-download"></i><span class="hidden lg:inline ml-1">Backup</span>
                        </button>
                        <button onclick="importData()" class="bg-green-600 hover:bg-green-700 text-white px-2 py-1 rounded text-xs" title="Khôi phục dữ liệu">
                            <i class="fas fa-upload"></i><span class="hidden lg:inline ml-1">Import</span>
                        </button>
                    </div>

                    <!-- User Info -->
                    <div class="flex items-center space-x-1 lg:space-x-2">
                        <span class="text-gray-600 text-sm hidden md:inline">Xin chào, <strong id="currentUser">Admin</strong></span>
                        <select id="userRole" class="bg-green-100 border border-green-300 rounded-lg px-2 lg:px-3 py-1 text-xs lg:text-sm" onchange="updatePermissions()">
                            <option value="admin">Admin</option>
                            <option value="director">Giám đốc</option>
                            <option value="lab_manager">TP PTN</option>
                            <option value="admin_staff">NV HC</option>
                            <option value="sampling_staff">NV QT</option>
                            <option value="lab_staff">NV PTN</option>
                            <option value="viewer">Xem</option>
                        </select>
                    </div>
                </div>
            </div>
        </div>
    </header>

    <div class="container mx-auto px-4 lg:px-6 py-4 lg:py-8 relative z-10">
        <!-- Mobile Menu Toggle -->
        <div class="lg:hidden mb-4">
            <button onclick="toggleMobileMenu()" class="w-full bg-white rounded-xl shadow-lg p-4 flex items-center justify-between">
                <span class="font-semibold text-gray-800">Menu</span>
                <i class="fas fa-bars text-gray-600" id="mobileMenuIcon"></i>
            </button>
        </div>

        <!-- Navigation Tabs -->
        <div class="bg-white rounded-xl shadow-lg mb-8 overflow-hidden">
            <div class="hidden lg:flex flex-wrap border-b overflow-x-auto lg:overflow-x-visible" id="tabContainer">
                <button onclick="showTab('dashboard')" class="tab-btn active px-3 lg:px-6 py-3 lg:py-4 font-semibold text-green-600 border-b-2 border-green-600 bg-green-50 whitespace-nowrap">
                    <i class="fas fa-tachometer-alt mr-1 lg:mr-2"></i><span class="hidden sm:inline">Tổng quan</span><span class="sm:hidden">TQ</span>
                </button>
                <button onclick="showTab('contracts')" class="tab-btn px-3 lg:px-6 py-3 lg:py-4 font-semibold text-gray-600 hover:text-green-600 hover:bg-green-50 transition-colors whitespace-nowrap">
                    <i class="fas fa-file-contract mr-1 lg:mr-2"></i><span class="hidden sm:inline">Hợp đồng</span><span class="sm:hidden">HĐ</span>
                </button>
                <button onclick="showTab('sampling')" class="tab-btn px-3 lg:px-6 py-3 lg:py-4 font-semibold text-gray-600 hover:text-green-600 hover:bg-green-50 transition-colors whitespace-nowrap">
                    <i class="fas fa-vial mr-1 lg:mr-2"></i><span class="hidden md:inline">Biên bản lấy mẫu</span><span class="md:hidden hidden sm:inline">Lấy mẫu</span><span class="sm:hidden">LM</span>
                </button>
                <button onclick="showTab('coding')" class="tab-btn px-3 lg:px-6 py-3 lg:py-4 font-semibold text-gray-600 hover:text-green-600 hover:bg-green-50 transition-colors whitespace-nowrap">
                    <i class="fas fa-barcode mr-1 lg:mr-2"></i><span class="hidden sm:inline">Mã hóa</span><span class="sm:hidden">MH</span>
                </button>
                <button onclick="showTab('analysis')" class="tab-btn px-3 lg:px-6 py-3 lg:py-4 font-semibold text-gray-600 hover:text-green-600 hover:bg-green-50 transition-colors whitespace-nowrap">
                    <i class="fas fa-microscope mr-1 lg:mr-2"></i><span class="hidden sm:inline">Phân tích</span><span class="sm:hidden">PT</span>
                </button>
                <button onclick="showTab('results')" class="tab-btn px-3 lg:px-6 py-3 lg:py-4 font-semibold text-gray-600 hover:text-green-600 hover:bg-green-50 transition-colors whitespace-nowrap">
                    <i class="fas fa-chart-line mr-1 lg:mr-2"></i><span class="hidden sm:inline">Kết quả</span><span class="sm:hidden">KQ</span>
                </button>
                <button onclick="showTab('customers')" class="tab-btn px-3 lg:px-6 py-3 lg:py-4 font-semibold text-gray-600 hover:text-green-600 hover:bg-green-50 transition-colors whitespace-nowrap">
                    <i class="fas fa-users mr-1 lg:mr-2"></i><span class="hidden sm:inline">Khách hàng</span><span class="sm:hidden">KH</span>
                </button>
                <button onclick="showTab('reports')" class="tab-btn px-3 lg:px-6 py-3 lg:py-4 font-semibold text-gray-600 hover:text-green-600 hover:bg-green-50 transition-colors whitespace-nowrap">
                    <i class="fas fa-chart-bar mr-1 lg:mr-2"></i><span class="hidden sm:inline">Báo cáo</span><span class="sm:hidden">BC</span>
                </button>
            </div>
        </div>

        <!-- Dashboard Tab -->
        <div id="dashboard" class="tab-content fade-in">
            <div class="grid grid-cols-2 lg:grid-cols-4 gap-3 lg:gap-6 mb-6 lg:mb-8">
                <div class="bg-gradient-to-r from-blue-500 to-blue-600 text-white p-3 lg:p-6 rounded-xl shadow-lg hover-scale">
                    <div class="flex flex-col lg:flex-row lg:items-center lg:justify-between">
                        <div class="mb-2 lg:mb-0">
                            <p class="text-blue-100 text-xs lg:text-sm">Hợp đồng đang thực hiện</p>
                            <p class="text-2xl lg:text-3xl font-bold" id="activeContracts">0</p>
                        </div>
                        <i class="fas fa-file-contract text-2xl lg:text-4xl text-blue-200 self-end lg:self-auto"></i>
                    </div>
                </div>
                <div class="bg-gradient-to-r from-green-500 to-green-600 text-white p-3 lg:p-6 rounded-xl shadow-lg hover-scale">
                    <div class="flex flex-col lg:flex-row lg:items-center lg:justify-between">
                        <div class="mb-2 lg:mb-0">
                            <p class="text-green-100 text-xs lg:text-sm">Mẫu nhận hôm nay</p>
                            <p class="text-2xl lg:text-3xl font-bold" id="todaySamples">0</p>
                        </div>
                        <i class="fas fa-vial text-2xl lg:text-4xl text-green-200 self-end lg:self-auto"></i>
                    </div>
                </div>
                <div class="bg-gradient-to-r from-orange-500 to-orange-600 text-white p-3 lg:p-6 rounded-xl shadow-lg hover-scale">
                    <div class="flex flex-col lg:flex-row lg:items-center lg:justify-between">
                        <div class="mb-2 lg:mb-0">
                            <p class="text-orange-100 text-xs lg:text-sm">Đang phân tích</p>
                            <p class="text-2xl lg:text-3xl font-bold" id="inAnalysis">0</p>
                        </div>
                        <i class="fas fa-microscope text-2xl lg:text-4xl text-orange-200 self-end lg:self-auto"></i>
                    </div>
                </div>
                <div class="bg-gradient-to-r from-purple-500 to-purple-600 text-white p-3 lg:p-6 rounded-xl shadow-lg hover-scale">
                    <div class="flex flex-col lg:flex-row lg:items-center lg:justify-between">
                        <div class="mb-2 lg:mb-0">
                            <p class="text-purple-100 text-xs lg:text-sm">Tổng file</p>
                            <p class="text-xl lg:text-2xl font-bold" id="totalFiles">0</p>
                        </div>
                        <i class="fas fa-folder text-2xl lg:text-4xl text-purple-200 self-end lg:self-auto"></i>
                    </div>
                </div>
            </div>

            <!-- Recent Activities -->
            <div class="bg-white rounded-xl shadow-lg p-6">
                <h3 class="text-xl font-bold text-gray-800 mb-4">
                    <i class="fas fa-clock mr-2 text-green-600"></i>Hoạt động gần đây
                </h3>
                <div class="space-y-4" id="recentActivities">
                    <p class="text-gray-500 text-center py-8">Chưa có hoạt động nào</p>
                </div>
            </div>
        </div>

        <!-- Contracts Tab -->
        <div id="contracts" class="tab-content hidden">
            <div class="bg-white rounded-xl shadow-lg p-6">
                <div class="flex justify-between items-center mb-6">
                    <h3 class="text-xl font-bold text-gray-800">
                        <i class="fas fa-file-contract mr-2 text-green-600"></i>Quản lý Hợp đồng
                    </h3>
                    <button onclick="openUploadModal('contracts')" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg transition-colors">
                        <i class="fas fa-upload mr-2"></i>Tải lên hợp đồng
                    </button>
                </div>
                
                <div class="overflow-x-auto">
                    <table class="w-full table-auto min-w-full">
                        <thead class="bg-gray-50">
                            <tr>
                                <th class="px-2 lg:px-4 py-3 text-left font-semibold text-gray-700 text-sm">Mã HĐ</th>
                                <th class="px-2 lg:px-4 py-3 text-left font-semibold text-gray-700 text-sm">Khách hàng</th>
                                <th class="px-2 lg:px-4 py-3 text-left font-semibold text-gray-700 text-sm hidden md:table-cell">Ngày ký</th>
                                <th class="px-2 lg:px-4 py-3 text-left font-semibold text-gray-700 text-sm">File</th>
                                <th class="px-2 lg:px-4 py-3 text-left font-semibold text-gray-700 text-sm">Thao tác</th>
                            </tr>
                        </thead>
                        <tbody class="divide-y divide-gray-200" id="contractsTable">
                            <tr><td colspan="5" class="text-center py-8 text-gray-500">Chưa có hợp đồng nào</td></tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- Sampling Tab -->
        <div id="sampling" class="tab-content hidden">
            <div class="bg-white rounded-xl shadow-lg p-6">
                <div class="flex justify-between items-center mb-6">
                    <h3 class="text-xl font-bold text-gray-800">
                        <i class="fas fa-vial mr-2 text-blue-600"></i>Biên bản Lấy mẫu
                    </h3>
                    <button onclick="openUploadModal('sampling')" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg transition-colors">
                        <i class="fas fa-upload mr-2"></i>Tải lên biên bản
                    </button>
                </div>
                
                <div class="overflow-x-auto">
                    <table class="w-full table-auto">
                        <thead class="bg-gray-50">
                            <tr>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Mã BB</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Khách hàng</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Ngày lấy mẫu</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">File đính kèm</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Thao tác</th>
                            </tr>
                        </thead>
                        <tbody class="divide-y divide-gray-200" id="samplingTable">
                            <tr><td colspan="5" class="text-center py-8 text-gray-500">Chưa có biên bản lấy mẫu nào</td></tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- Coding Tab -->
        <div id="coding" class="tab-content hidden">
            <div class="bg-white rounded-xl shadow-lg p-6">
                <div class="flex justify-between items-center mb-6">
                    <h3 class="text-xl font-bold text-gray-800">
                        <i class="fas fa-barcode mr-2 text-blue-600"></i>Danh mục Mã hóa Mẫu
                    </h3>
                    <button onclick="openUploadModal('coding')" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg transition-colors">
                        <i class="fas fa-upload mr-2"></i>Tải lên danh mục
                    </button>
                </div>
                
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4" id="codingGrid">
                    <div class="col-span-3 text-center py-8 text-gray-500">Chưa có danh mục mã hóa nào</div>
                </div>
            </div>
        </div>

        <!-- Analysis Tab -->
        <div id="analysis" class="tab-content hidden">
            <div class="bg-white rounded-xl shadow-lg p-6">
                <div class="flex justify-between items-center mb-6">
                    <h3 class="text-xl font-bold text-gray-800">
                        <i class="fas fa-microscope mr-2 text-blue-600"></i>Biên bản Phân tích
                    </h3>
                    <button onclick="openUploadModal('analysis')" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg transition-colors">
                        <i class="fas fa-upload mr-2"></i>Tải lên biên bản
                    </button>
                </div>
                
                <div class="overflow-x-auto">
                    <table class="w-full table-auto">
                        <thead class="bg-gray-50">
                            <tr>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Mã phân tích</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Khách hàng</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Ngày phân tích</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">File đính kèm</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Thao tác</th>
                            </tr>
                        </thead>
                        <tbody class="divide-y divide-gray-200" id="analysisTable">
                            <tr><td colspan="5" class="text-center py-8 text-gray-500">Chưa có biên bản phân tích nào</td></tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- Results Tab -->
        <div id="results" class="tab-content hidden">
            <div class="bg-white rounded-xl shadow-lg p-6">
                <div class="flex justify-between items-center mb-6">
                    <h3 class="text-xl font-bold text-gray-800">
                        <i class="fas fa-chart-line mr-2 text-blue-600"></i>Phiếu Kết quả Thử nghiệm
                    </h3>
                    <button onclick="openUploadModal('results')" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg transition-colors">
                        <i class="fas fa-upload mr-2"></i>Tải lên kết quả
                    </button>
                </div>
                
                <div class="overflow-x-auto">
                    <table class="w-full table-auto">
                        <thead class="bg-gray-50">
                            <tr>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Mã phiếu</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Khách hàng</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Ngày hoàn thành</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">File đính kèm</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Thao tác</th>
                            </tr>
                        </thead>
                        <tbody class="divide-y divide-gray-200" id="resultsTable">
                            <tr><td colspan="5" class="text-center py-8 text-gray-500">Chưa có kết quả thử nghiệm nào</td></tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- Customers Tab -->
        <div id="customers" class="tab-content hidden">
            <div class="bg-white rounded-xl shadow-lg p-6">
                <div class="flex justify-between items-center mb-6">
                    <h3 class="text-xl font-bold text-gray-800">
                        <i class="fas fa-users mr-2 text-blue-600"></i>Quản lý Khách hàng
                    </h3>
                </div>
                
                <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                    <div>
                        <h4 class="font-semibold text-gray-700 mb-4">Danh sách khách hàng</h4>
                        <div class="space-y-3" id="customerList">
                            <p class="text-gray-500">Chưa có khách hàng nào</p>
                        </div>
                    </div>
                    
                    <div>
                        <h4 class="font-semibold text-gray-700 mb-4">Tiến độ theo dõi</h4>
                        <div id="customerProgress" class="bg-gray-50 rounded-lg p-4 text-center text-gray-500">
                            Chọn khách hàng để xem tiến độ
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Reports Tab -->
        <div id="reports" class="tab-content hidden">
            <div class="bg-white rounded-xl shadow-lg p-6">
                <h3 class="text-xl font-bold text-gray-800 mb-6">
                    <i class="fas fa-chart-bar mr-2 text-blue-600"></i>Báo cáo Thống kê
                </h3>
                
                <div class="grid grid-cols-2 lg:grid-cols-4 gap-4 mb-6">
                    <div class="bg-blue-50 p-4 rounded-lg text-center">
                        <i class="fas fa-file-contract text-2xl text-blue-600 mb-2"></i>
                        <p class="text-sm text-gray-600">Tổng hợp đồng</p>
                        <p class="text-xl font-bold text-blue-600" id="reportContracts">0</p>
                    </div>
                    <div class="bg-green-50 p-4 rounded-lg text-center">
                        <i class="fas fa-vial text-2xl text-green-600 mb-2"></i>
                        <p class="text-sm text-gray-600">Tổng mẫu</p>
                        <p class="text-xl font-bold text-green-600" id="reportSamples">0</p>
                    </div>
                    <div class="bg-orange-50 p-4 rounded-lg text-center">
                        <i class="fas fa-microscope text-2xl text-orange-600 mb-2"></i>
                        <p class="text-sm text-gray-600">Phân tích</p>
                        <p class="text-xl font-bold text-orange-600" id="reportAnalysis">0</p>
                    </div>
                    <div class="bg-purple-50 p-4 rounded-lg text-center">
                        <i class="fas fa-chart-line text-2xl text-purple-600 mb-2"></i>
                        <p class="text-sm text-gray-600">Kết quả</p>
                        <p class="text-xl font-bold text-purple-600" id="reportResults">0</p>
                    </div>
                </div>
                
                <div class="text-center py-8 text-gray-500">
                    <i class="fas fa-chart-bar text-4xl mb-4"></i>
                    <p>Báo cáo chi tiết sẽ được cập nhật khi có dữ liệu</p>
                </div>
            </div>
        </div>
    </div>

    <!-- Upload Modal -->
    <div id="uploadModal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl w-full max-w-2xl max-h-full overflow-auto">
            <div class="flex justify-between items-center p-6 border-b">
                <h3 id="uploadModalTitle" class="text-lg font-bold">Tải lên file</h3>
                <button onclick="closeModal('uploadModal')" class="text-gray-500 hover:text-gray-700">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            <div class="p-6">
                <form id="uploadForm">
                    <div class="mb-4">
                        <label class="block text-sm font-medium text-gray-700 mb-2">Khách hàng</label>
                        <div class="flex space-x-2">
                            <select id="uploadCustomer" class="flex-1 border border-gray-300 rounded-lg px-3 py-2" required>
                                <option value="">Chọn khách hàng</option>
                            </select>
                            <button type="button" onclick="addNewCustomer()" class="bg-green-600 hover:bg-green-700 text-white px-3 py-2 rounded-lg" title="Thêm khách hàng mới">
                                <i class="fas fa-plus"></i>
                            </button>
                        </div>
                        <input type="text" id="newCustomerInput" class="w-full border border-gray-300 rounded-lg px-3 py-2 mt-2 hidden" placeholder="Nhập tên khách hàng mới...">
                    </div>
                    
                    <div class="mb-4">
                        <label class="block text-sm font-medium text-gray-700 mb-2">Mã tham chiếu</label>
                        <input type="text" id="uploadReference" class="w-full border border-gray-300 rounded-lg px-3 py-2" placeholder="Ví dụ: HD001, BB001, MA001..." required>
                    </div>
                    
                    <div class="mb-6">
                        <label class="block text-sm font-medium text-gray-700 mb-2">File đính kèm</label>
                        <div id="dropZone" class="drag-drop-zone border-2 border-dashed border-gray-300 rounded-lg p-8 text-center cursor-pointer hover:border-blue-400 hover:bg-blue-50 transition-colors">
                            <i class="fas fa-cloud-upload-alt text-4xl text-gray-400 mb-4"></i>
                            <p class="text-gray-600 mb-2">Kéo thả file vào đây hoặc click để chọn</p>
                            <p class="text-sm text-gray-500">Hỗ trợ: Word, Excel, PDF, Hình ảnh (tối đa 10MB)</p>
                            <input type="file" id="fileInput" class="hidden" multiple accept=".doc,.docx,.xls,.xlsx,.pdf,.jpg,.jpeg,.png,.gif">
                        </div>
                        <div id="fileList" class="mt-4 space-y-2"></div>
                    </div>
                    
                    <div class="flex justify-end space-x-4">
                        <button type="button" onclick="closeModal('uploadModal')" class="bg-gray-500 hover:bg-gray-600 text-white px-4 py-2 rounded-lg">
                            Hủy
                        </button>
                        <button type="submit" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg">
                            <i class="fas fa-upload mr-2"></i>Tải lên
                        </button>
                    </div>
                </form>
            </div>
        </div>
    </div>

    <!-- File Preview Modal -->
    <div id="previewModal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50 p-4">
        <div class="bg-white rounded-xl w-full max-w-4xl h-full lg:h-5/6 flex flex-col">
            <div class="flex justify-between items-center p-6 border-b">
                <div>
                    <h3 id="previewTitle" class="text-lg font-bold">Preview File</h3>
                    <p id="previewSubtitle" class="text-sm text-gray-600">File information</p>
                </div>
                <div class="flex space-x-2">
                    <button onclick="downloadFile(currentPreviewFile)" class="bg-green-600 hover:bg-green-700 text-white px-3 py-2 rounded-lg text-sm">
                        <i class="fas fa-download mr-1"></i>Tải xuống
                    </button>
                    <button onclick="closeModal('previewModal')" class="text-gray-500 hover:text-gray-700">
                        <i class="fas fa-times text-xl"></i>
                    </button>
                </div>
            </div>
            <div class="flex-1 p-6 overflow-auto preview-container">
                <div id="previewContent" class="h-full">
                    <!-- Preview content will be loaded here -->
                </div>
            </div>
        </div>
    </div>

    <script>
        // Global variables
        let currentPreviewFile = null;
        let currentUploadType = '';
        let selectedFiles = [];
        let fileDatabase = {
            contracts: [],
            sampling: [],
            coding: [],
            analysis: [],
            results: []
        };
        let customerList = [];

        // Permission system
        const permissions = {
            admin: ['dashboard', 'contracts', 'sampling', 'coding', 'analysis', 'results', 'customers', 'reports'],
            director: ['dashboard', 'contracts', 'sampling', 'coding', 'analysis', 'results', 'customers', 'reports'],
            lab_manager: ['dashboard', 'analysis', 'results', 'customers', 'reports'],
            admin_staff: ['dashboard', 'contracts', 'customers', 'reports'],
            sampling_staff: ['dashboard', 'sampling', 'coding', 'customers', 'reports'],
            lab_staff: ['dashboard', 'analysis', 'customers', 'reports'],
            viewer: ['dashboard', 'customers', 'reports']
        };

        // Initialize app
        document.addEventListener('DOMContentLoaded', function() {
            loadFromLocal();
            setupDragAndDrop();
            setupUploadForm();
            loadDashboard();
            updatePermissions();
        });

        // Local storage functions
        function saveToLocal() {
            try {
                localStorage.setItem('cemHueLabData', JSON.stringify(fileDatabase));
                localStorage.setItem('cemHueCustomers', JSON.stringify(customerList));
                localStorage.setItem('cemHueLabLastUpdate', new Date().toISOString());
                console.log('✅ Đã lưu dữ liệu vào localStorage');
            } catch (error) {
                console.error('❌ Lỗi lưu dữ liệu:', error);
                showNotification('Lỗi lưu dữ liệu!', 'error');
            }
        }

        function loadFromLocal() {
            try {
                const saved = localStorage.getItem('cemHueLabData');
                const savedCustomers = localStorage.getItem('cemHueCustomers');
                
                if (saved) {
                    fileDatabase = JSON.parse(saved);
                    console.log('✅ Đã tải dữ liệu từ localStorage');
                }
                
                if (savedCustomers) {
                    customerList = JSON.parse(savedCustomers);
                } else {
                    customerList = [];
                }
                
                loadCustomerOptions();
            } catch (error) {
                console.error('❌ Lỗi tải dữ liệu:', error);
                fileDatabase = {
                    contracts: [],
                    sampling: [],
                    coding: [],
                    analysis: [],
                    results: []
                };
                customerList = [];
            }
        }

        // Notification system
        function showNotification(message, type = 'success') {
            const notification = document.createElement('div');
            notification.className = `notification ${type}`;
            notification.innerHTML = `
                <div class="flex items-center">
                    <i class="fas ${type === 'success' ? 'fa-check-circle' : type === 'error' ? 'fa-times-circle' : type === 'warning' ? 'fa-exclamation-triangle' : 'fa-info-circle'} mr-2"></i>
                    <span>${message}</span>
                </div>
            `;
            
            document.body.appendChild(notification);
            
            // Show notification
            setTimeout(() => notification.classList.add('show'), 100);
            
            // Hide notification
            setTimeout(() => {
                notification.classList.remove('show');
                setTimeout(() => document.body.removeChild(notification), 300);
            }, 3000);
        }

        // Mobile menu toggle
        let mobileMenuOpen = false;
        function toggleMobileMenu() {
            const tabContainer = document.getElementById('tabContainer');
            const icon = document.getElementById('mobileMenuIcon');
            
            mobileMenuOpen = !mobileMenuOpen;
            
            if (mobileMenuOpen) {
                tabContainer.classList.remove('hidden');
                tabContainer.classList.add('flex', 'flex-col', 'lg:flex-row');
                icon.className = 'fas fa-times text-gray-600';
            } else {
                tabContainer.classList.add('hidden', 'lg:flex');
                tabContainer.classList.remove('flex', 'flex-col');
                icon.className = 'fas fa-bars text-gray-600';
            }
        }

        // Tab switching
        function showTab(tabName) {
            const userRole = document.getElementById('userRole').value;
            if (!hasPermission(tabName, userRole)) {
                showNotification('Bạn không có quyền truy cập mục này!', 'warning');
                return;
            }

            // Hide all tabs
            document.querySelectorAll('.tab-content').forEach(tab => {
                tab.classList.add('hidden');
            });
            
            // Remove active class from all buttons
            document.querySelectorAll('.tab-btn').forEach(btn => {
                btn.classList.remove('active', 'text-green-600', 'border-b-2', 'border-green-600', 'bg-green-50');
                btn.classList.add('text-gray-600');
            });
            
            // Show selected tab
            const selectedTab = document.getElementById(tabName);
            if (selectedTab) {
                selectedTab.classList.remove('hidden');
                selectedTab.classList.add('fade-in');
            }
            
            // Activate button
            const activeBtn = document.querySelector(`button[onclick="showTab('${tabName}')"]`);
            if (activeBtn) {
                activeBtn.classList.add('active', 'text-green-600', 'border-b-2', 'border-green-600', 'bg-green-50');
                activeBtn.classList.remove('text-gray-600');
            }

            // Load tab-specific data
            loadTabData(tabName);
        }

        function hasPermission(tabName, userRole) {
            return permissions[userRole]?.includes(tabName) || false;
        }

        function updatePermissions() {
            const role = document.getElementById('userRole').value;
            
            // Update tab visibility
            document.querySelectorAll('.tab-btn').forEach(btn => {
                const tabName = btn.getAttribute('onclick')?.match(/showTab\('(.+?)'\)/)?.[1];
                if (tabName && !hasPermission(tabName, role)) {
                    btn.style.opacity = '0.5';
                    btn.style.pointerEvents = 'none';
                } else {
                    btn.style.opacity = '1';
                    btn.style.pointerEvents = 'auto';
                }
            });

            // Update user name
            const roleNames = {
                admin: 'Admin Hệ thống',
                director: 'Giám đốc',
                lab_manager: 'Trưởng phòng PTN',
                admin_staff: 'NV Hành chính',
                sampling_staff: 'NV Quan trắc',
                lab_staff: 'NV Phòng thí nghiệm',
                viewer: 'Người xem'
            };
            
            document.getElementById('currentUser').textContent = roleNames[role];
        }

        // Upload modal functions
        function openUploadModal(type) {
            currentUploadType = type;
            selectedFiles = [];
            
            const titles = {
                contracts: 'Tải lên Hợp đồng',
                sampling: 'Tải lên Biên bản Lấy mẫu',
                coding: 'Tải lên Danh mục Mã hóa',
                analysis: 'Tải lên Biên bản Phân tích',
                results: 'Tải lên Kết quả Thử nghiệm'
            };
            
            document.getElementById('uploadModalTitle').textContent = titles[type];
            document.getElementById('uploadModal').classList.remove('hidden');
            document.getElementById('uploadModal').classList.add('flex');
        }

        function closeModal(modalId) {
            document.getElementById(modalId).classList.add('hidden');
            document.getElementById(modalId).classList.remove('flex');
            
            // Reset forms
            if (modalId === 'uploadModal') {
                document.getElementById('uploadForm').reset();
                document.getElementById('fileList').innerHTML = '';
                selectedFiles = [];
            }
        }

        // Drag and drop setup
        function setupDragAndDrop() {
            const dropZone = document.getElementById('dropZone');
            const fileInput = document.getElementById('fileInput');
            
            if (!dropZone || !fileInput) return;
            
            dropZone.addEventListener('click', () => fileInput.click());
            
            dropZone.addEventListener('dragover', (e) => {
                e.preventDefault();
                dropZone.classList.add('dragover');
            });
            
            dropZone.addEventListener('dragleave', () => {
                dropZone.classList.remove('dragover');
            });
            
            dropZone.addEventListener('drop', (e) => {
                e.preventDefault();
                dropZone.classList.remove('dragover');
                handleFiles(e.dataTransfer.files);
            });
            
            fileInput.addEventListener('change', (e) => {
                handleFiles(e.target.files);
            });
        }

        function handleFiles(files) {
            const fileList = document.getElementById('fileList');
            
            Array.from(files).forEach((file, index) => {
                if (file.size > 10 * 1024 * 1024) { // 10MB limit
                    showNotification(`File ${file.name} quá lớn (>10MB)`, 'warning');
                    return;
                }
                
                selectedFiles.push(file);
                const fileItem = createFileItem(file, selectedFiles.length - 1);
                fileList.appendChild(fileItem);
            });
        }

        function createFileItem(file, index) {
            const div = document.createElement('div');
            div.className = 'flex items-center justify-between p-3 bg-gray-50 rounded-lg';
            div.id = `file-item-${index}`;
            
            const fileType = getFileType(file.name);
            const iconClass = getFileIcon(fileType);
            
            div.innerHTML = `
                <div class="flex items-center space-x-3">
                    <div class="file-type-icon ${fileType}-icon">
                        <i class="${iconClass}"></i>
                    </div>
                    <div>
                        <p class="font-medium text-gray-800">${file.name}</p>
                        <p class="text-sm text-gray-500">${formatFileSize(file.size)}</p>
                    </div>
                </div>
                <div class="flex items-center space-x-2">
                    <div class="w-32 bg-gray-200 rounded-full h-2">
                        <div class="bg-blue-600 h-2 rounded-full progress-bar" style="width: 0%"></div>
                    </div>
                    <button onclick="removeFile(${index})" class="text-red-600 hover:text-red-800">
                        <i class="fas fa-times"></i>
                    </button>
                </div>
            `;
            
            return div;
        }

        function removeFile(index) {
            selectedFiles.splice(index, 1);
            const fileItem = document.getElementById(`file-item-${index}`);
            if (fileItem) {
                fileItem.remove();
            }
            
            // Re-render file list with updated indices
            const fileList = document.getElementById('fileList');
            fileList.innerHTML = '';
            selectedFiles.forEach((file, newIndex) => {
                const fileItem = createFileItem(file, newIndex);
                fileList.appendChild(fileItem);
            });
        }

        function getFileType(filename) {
            const ext = filename.split('.').pop().toLowerCase();
            if (['doc', 'docx'].includes(ext)) return 'word';
            if (['xls', 'xlsx'].includes(ext)) return 'excel';
            if (ext === 'pdf') return 'pdf';
            if (['jpg', 'jpeg', 'png', 'gif'].includes(ext)) return 'image';
            return 'file';
        }

        function getFileIcon(type) {
            const icons = {
                word: 'fas fa-file-word',
                excel: 'fas fa-file-excel',
                pdf: 'fas fa-file-pdf',
                image: 'fas fa-file-image',
                file: 'fas fa-file'
            };
            return icons[type] || 'fas fa-file';
        }

        function formatFileSize(bytes) {
            if (bytes === 0) return '0 Bytes';
            const k = 1024;
            const sizes = ['Bytes', 'KB', 'MB', 'GB'];
            const i = Math.floor(Math.log(bytes) / Math.log(k));
            return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
        }

        // Upload form setup
        function setupUploadForm() {
            const form = document.getElementById('uploadForm');
            if (!form) return;
            
            form.addEventListener('submit', async function(e) {
                e.preventDefault();
                
                const customer = document.getElementById('uploadCustomer').value;
                const reference = document.getElementById('uploadReference').value;
                
                if (!customer || !reference || selectedFiles.length === 0) {
                    showNotification('Vui lòng điền đầy đủ thông tin!', 'warning');
                    return;
                }
                
                try {
                    await simulateUpload(selectedFiles, customer, reference);
                    closeModal('uploadModal');
                    loadTabData(currentUploadType);
                    showNotification('Tải lên thành công!', 'success');
                } catch (error) {
                    console.error('Upload error:', error);
                    showNotification('Lỗi tải lên file!', 'error');
                }
            });
        }

        async function simulateUpload(files, customer, reference) {
            const progressBars = document.querySelectorAll('.progress-bar');
            
            for (let i = 0; i < files.length; i++) {
                const file = files[i];
                const progressBar = progressBars[i];
                
                if (progressBar) {
                    // Simulate upload progress
                    for (let progress = 0; progress <= 100; progress += 20) {
                        progressBar.style.width = progress + '%';
                        await new Promise(resolve => setTimeout(resolve, 100));
                    }
                }
                
                // Convert file to base64 for local storage
                const base64Data = await fileToBase64(file);
                
                // Add to database
                const fileData = {
                    id: Date.now() + i,
                    name: file.name,
                    size: file.size,
                    type: getFileType(file.name),
                    customer: customer,
                    reference: reference,
                    uploadDate: new Date().toISOString(),
                    localData: base64Data
                };
                
                fileDatabase[currentUploadType].push(fileData);
            }
            
            // Save to localStorage
            saveToLocal();
        }

        function fileToBase64(file) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.readAsDataURL(file);
                reader.onload = () => resolve(reader.result);
                reader.onerror = error => reject(error);
            });
        }

        // File operations
        function previewFile(fileData) {
            currentPreviewFile = fileData;
            
            document.getElementById('previewTitle').textContent = fileData.name;
            document.getElementById('previewSubtitle').textContent = `${fileData.customer} - ${formatFileSize(fileData.size)}`;
            
            const previewContent = document.getElementById('previewContent');
            previewContent.innerHTML = generatePreview(fileData);
            
            document.getElementById('previewModal').classList.remove('hidden');
            document.getElementById('previewModal').classList.add('flex');
        }

        function generatePreview(fileData) {
            return `
                <div class="bg-white border rounded-lg p-6 shadow-sm">
                    <div class="flex items-center mb-4">
                        <i class="fas ${getFileIcon(fileData.type)} text-2xl mr-3 ${getFileColor(fileData.type)}"></i>
                        <div>
                            <h4 class="font-bold text-gray-800">${fileData.name}</h4>
                            <p class="text-sm text-gray-600">${fileData.customer} - ${fileData.reference}</p>
                        </div>
                    </div>
                    <div class="bg-gray-50 p-4 rounded border-l-4 border-blue-500">
                        <h5 class="font-semibold mb-2">Thông tin file:</h5>
                        <p class="text-gray-700 leading-relaxed">
                            <strong>Khách hàng:</strong> ${fileData.customer}<br>
                            <strong>Mã tham chiếu:</strong> ${fileData.reference}<br>
                            <strong>Ngày tải lên:</strong> ${new Date(fileData.uploadDate).toLocaleDateString('vi-VN')}<br>
                            <strong>Kích thước:</strong> ${formatFileSize(fileData.size)}<br>
                            <strong>Loại file:</strong> ${fileData.type.toUpperCase()}<br><br>
                            <em>Đây là preview mẫu cho file đã tải lên...</em>
                        </p>
                    </div>
                </div>
            `;
        }

        function getFileColor(type) {
            const colors = {
                word: 'text-blue-600',
                excel: 'text-green-600',
                pdf: 'text-red-600',
                image: 'text-purple-600'
            };
            return colors[type] || 'text-gray-600';
        }

        function downloadFile(fileData) {
            if (fileData.localData) {
                const link = document.createElement('a');
                link.href = fileData.localData;
                link.download = fileData.name;
                link.click();
                showNotification(`Đã tải xuống ${fileData.name}`, 'success');
            } else {
                showNotification(`File ${fileData.name} không có dữ liệu`, 'error');
            }
        }

        function deleteFile(fileData, type) {
            if (confirm(`Bạn có chắc muốn xóa file ${fileData.name}?`)) {
                const index = fileDatabase[type].findIndex(f => f.id === fileData.id);
                if (index > -1) {
                    fileDatabase[type].splice(index, 1);
                    saveToLocal();
                    loadTabData(type);
                    showNotification(`Đã xóa ${fileData.name}`, 'success');
                }
            }
        }

        // Load tab data
        function loadTabData(tabName) {
            switch (tabName) {
                case 'dashboard':
                    loadDashboard();
                    break;
                case 'contracts':
                    loadContracts();
                    break;
                case 'sampling':
                    loadSampling();
                    break;
                case 'coding':
                    loadCoding();
                    break;
                case 'analysis':
                    loadAnalysis();
                    break;
                case 'results':
                    loadResults();
                    break;
                case 'customers':
                    loadCustomers();
                    break;
                case 'reports':
                    loadReports();
                    break;
            }
        }

        function loadDashboard() {
            // Update statistics
            document.getElementById('activeContracts').textContent = fileDatabase.contracts.length;
            document.getElementById('todaySamples').textContent = getTodaySamplesCount();
            document.getElementById('inAnalysis').textContent = fileDatabase.analysis.length;
            document.getElementById('totalFiles').textContent = getTotalFilesCount();
            document.getElementById('dailySamples').textContent = getTodaySamplesCount();
            
            // Load recent activities
            loadRecentActivities();
        }

        function getTodaySamplesCount() {
            const today = new Date().toDateString();
            return fileDatabase.sampling.filter(s => 
                new Date(s.uploadDate).toDateString() === today
            ).length;
        }

        function getTotalFilesCount() {
            return Object.values(fileDatabase).reduce((sum, arr) => sum + arr.length, 0);
        }

        function loadRecentActivities() {
            const container = document.getElementById('recentActivities');
            const allFiles = [
                ...fileDatabase.contracts.map(f => ({...f, type: 'contracts', icon: 'fa-file-contract', color: 'blue'})),
                ...fileDatabase.sampling.map(f => ({...f, type: 'sampling', icon: 'fa-vial', color: 'green'})),
                ...fileDatabase.analysis.map(f => ({...f, type: 'analysis', icon: 'fa-microscope', color: 'orange'})),
                ...fileDatabase.results.map(f => ({...f, type: 'results', icon: 'fa-chart-line', color: 'purple'}))
            ].sort((a, b) => new Date(b.uploadDate) - new Date(a.uploadDate)).slice(0, 5);
            
            if (allFiles.length === 0) {
                container.innerHTML = '<p class="text-gray-500 text-center py-8">Chưa có hoạt động nào</p>';
                return;
            }
            
            container.innerHTML = allFiles.map(file => `
                <div class="flex items-center p-4 bg-${file.color}-50 rounded-lg hover:bg-${file.color}-100 transition-colors">
                    <i class="fas ${file.icon} text-${file.color}-600 mr-3"></i>
                    <div class="flex-1">
                        <p class="font-semibold">Tải lên ${file.name}</p>
                        <p class="text-sm text-gray-600">${file.customer} - ${file.reference}</p>
                    </div>
                    <span class="text-sm text-gray-500">${getTimeAgo(file.uploadDate)}</span>
                </div>
            `).join('');
        }

        function getTimeAgo(dateString) {
            const now = new Date();
            const date = new Date(dateString);
            const diffMs = now - date;
            const diffMins = Math.floor(diffMs / 60000);
            const diffHours = Math.floor(diffMs / 3600000);
            const diffDays = Math.floor(diffMs / 86400000);
            
            if (diffMins < 60) return `${diffMins} phút trước`;
            if (diffHours < 24) return `${diffHours} giờ trước`;
            return `${diffDays} ngày trước`;
        }

        function loadContracts() {
            const tbody = document.getElementById('contractsTable');
            
            if (fileDatabase.contracts.length === 0) {
                tbody.innerHTML = '<tr><td colspan="5" class="text-center py-8 text-gray-500">Chưa có hợp đồng nào</td></tr>';
                return;
            }
            
            tbody.innerHTML = fileDatabase.contracts.map(contract => `
                <tr class="hover:bg-gray-50">
                    <td class="px-4 py-3 font-medium">${contract.reference}</td>
                    <td class="px-4 py-3">${contract.customer}</td>
                    <td class="px-4 py-3 hidden md:table-cell">${new Date(contract.uploadDate).toLocaleDateString('vi-VN')}</td>
                    <td class="px-4 py-3">
                        <div class="flex items-center space-x-2">
                            <i class="fas ${getFileIcon(contract.type)} ${getFileColor(contract.type)}"></i>
                            <span class="truncate max-w-xs">${contract.name}</span>
                        </div>
                    </td>
                    <td class="px-4 py-3">
                        <div class="flex space-x-1">
                            <button onclick="previewFile(${JSON.stringify(contract).replace(/"/g, '&quot;')})" class="text-blue-600 hover:text-blue-800 p-1" title="Xem trước">
                                <i class="fas fa-eye"></i>
                            </button>
                            <button onclick="downloadFile(${JSON.stringify(contract).replace(/"/g, '&quot;')})" class="text-green-600 hover:text-green-800 p-1" title="Tải xuống">
                                <i class="fas fa-download"></i>
                            </button>
                            <button onclick="deleteFile(${JSON.stringify(contract).replace(/"/g, '&quot;')}, 'contracts')" class="text-red-600 hover:text-red-800 p-1" title="Xóa">
                                <i class="fas fa-trash"></i>
                            </button>
                        </div>
                    </td>
                </tr>
            `).join('');
        }

        function loadSampling() {
            const tbody = document.getElementById('samplingTable');
            
            if (fileDatabase.sampling.length === 0) {
                tbody.innerHTML = '<tr><td colspan="5" class="text-center py-8 text-gray-500">Chưa có biên bản lấy mẫu nào</td></tr>';
                return;
            }
            
            tbody.innerHTML = fileDatabase.sampling.map(sample => `
                <tr class="hover:bg-gray-50">
                    <td class="px-4 py-3 font-medium">${sample.reference}</td>
                    <td class="px-4 py-3">${sample.customer}</td>
                    <td class="px-4 py-3">${new Date(sample.uploadDate).toLocaleDateString('vi-VN')}</td>
                    <td class="px-4 py-3">
                        <div class="flex items-center space-x-2">
                            <i class="fas ${getFileIcon(sample.type)} ${getFileColor(sample.type)}"></i>
                            <span class="truncate max-w-xs">${sample.name}</span>
                        </div>
                    </td>
                    <td class="px-4 py-3">
                        <div class="flex space-x-1">
                            <button onclick="previewFile(${JSON.stringify(sample).replace(/"/g, '&quot;')})" class="text-blue-600 hover:text-blue-800 p-1" title="Xem trước">
                                <i class="fas fa-eye"></i>
                            </button>
                            <button onclick="downloadFile(${JSON.stringify(sample).replace(/"/g, '&quot;')})" class="text-green-600 hover:text-green-800 p-1" title="Tải xuống">
                                <i class="fas fa-download"></i>
                            </button>
                            <button onclick="deleteFile(${JSON.stringify(sample).replace(/"/g, '&quot;')}, 'sampling')" class="text-red-600 hover:text-red-800 p-1" title="Xóa">
                                <i class="fas fa-trash"></i>
                            </button>
                        </div>
                    </td>
                </tr>
            `).join('');
        }

        function loadCoding() {
            const container = document.getElementById('codingGrid');
            
            if (fileDatabase.coding.length === 0) {
                container.innerHTML = '<div class="col-span-3 text-center py-8 text-gray-500">Chưa có danh mục mã hóa nào</div>';
                return;
            }
            
            container.innerHTML = fileDatabase.coding.map(code => `
                <div class="border border-gray-200 rounded-lg p-4 hover:shadow-md transition-shadow">
                    <div class="flex items-center justify-between mb-2">
                        <span class="font-bold text-lg text-blue-600">${code.reference}</span>
                        <span class="bg-green-100 text-green-800 px-2 py-1 rounded text-sm">Đã mã hóa</span>
                    </div>
                    <p class="text-gray-600 mb-2">${code.customer}</p>
                    <p class="text-sm text-gray-500 mb-3">${new Date(code.uploadDate).toLocaleDateString('vi-VN')}</p>
                    <div class="flex space-x-2">
                        <button onclick="previewFile(${JSON.stringify(code).replace(/"/g, '&quot;')})" class="text-blue-600 hover:text-blue-800" title="Xem trước">
                            <i class="fas fa-eye"></i>
                        </button>
                        <button onclick="downloadFile(${JSON.stringify(code).replace(/"/g, '&quot;')})" class="text-green-600 hover:text-green-800" title="Tải xuống">
                            <i class="fas fa-download"></i>
                        </button>
                        <button onclick="deleteFile(${JSON.stringify(code).replace(/"/g, '&quot;')}, 'coding')" class="text-red-600 hover:text-red-800" title="Xóa">
                            <i class="fas fa-trash"></i>
                        </button>
                    </div>
                </div>
            `).join('');
        }

        function loadAnalysis() {
            const tbody = document.getElementById('analysisTable');
            
            if (fileDatabase.analysis.length === 0) {
                tbody.innerHTML = '<tr><td colspan="5" class="text-center py-8 text-gray-500">Chưa có biên bản phân tích nào</td></tr>';
                return;
            }
            
            tbody.innerHTML = fileDatabase.analysis.map(analysis => `
                <tr class="hover:bg-gray-50">
                    <td class="px-4 py-3 font-medium">${analysis.reference}</td>
                    <td class="px-4 py-3">${analysis.customer}</td>
                    <td class="px-4 py-3">${new Date(analysis.uploadDate).toLocaleDateString('vi-VN')}</td>
                    <td class="px-4 py-3">
                        <div class="flex items-center space-x-2">
                            <i class="fas ${getFileIcon(analysis.type)} ${getFileColor(analysis.type)}"></i>
                            <span class="truncate max-w-xs">${analysis.name}</span>
                        </div>
                    </td>
                    <td class="px-4 py-3">
                        <div class="flex space-x-1">
                            <button onclick="previewFile(${JSON.stringify(analysis).replace(/"/g, '&quot;')})" class="text-blue-600 hover:text-blue-800 p-1" title="Xem trước">
                                <i class="fas fa-eye"></i>
                            </button>
                            <button onclick="downloadFile(${JSON.stringify(analysis).replace(/"/g, '&quot;')})" class="text-green-600 hover:text-green-800 p-1" title="Tải xuống">
                                <i class="fas fa-download"></i>
                            </button>
                            <button onclick="deleteFile(${JSON.stringify(analysis).replace(/"/g, '&quot;')}, 'analysis')" class="text-red-600 hover:text-red-800 p-1" title="Xóa">
                                <i class="fas fa-trash"></i>
                            </button>
                        </div>
                    </td>
                </tr>
            `).join('');
        }

        function loadResults() {
            const tbody = document.getElementById('resultsTable');
            
            if (fileDatabase.results.length === 0) {
                tbody.innerHTML = '<tr><td colspan="5" class="text-center py-8 text-gray-500">Chưa có kết quả thử nghiệm nào</td></tr>';
                return;
            }
            
            tbody.innerHTML = fileDatabase.results.map(result => `
                <tr class="hover:bg-gray-50">
                    <td class="px-4 py-3 font-medium">${result.reference}</td>
                    <td class="px-4 py-3">${result.customer}</td>
                    <td class="px-4 py-3">${new Date(result.uploadDate).toLocaleDateString('vi-VN')}</td>
                    <td class="px-4 py-3">
                        <div class="flex items-center space-x-2">
                            <i class="fas ${getFileIcon(result.type)} ${getFileColor(result.type)}"></i>
                            <span class="truncate max-w-xs">${result.name}</span>
                        </div>
                    </td>
                    <td class="px-4 py-3">
                        <div class="flex space-x-1">
                            <button onclick="previewFile(${JSON.stringify(result).replace(/"/g, '&quot;')})" class="text-blue-600 hover:text-blue-800 p-1" title="Xem trước">
                                <i class="fas fa-eye"></i>
                            </button>
                            <button onclick="downloadFile(${JSON.stringify(result).replace(/"/g, '&quot;')})" class="text-green-600 hover:text-green-800 p-1" title="Tải xuống">
                                <i class="fas fa-download"></i>
                            </button>
                            <button onclick="deleteFile(${JSON.stringify(result).replace(/"/g, '&quot;')}, 'results')" class="text-red-600 hover:text-red-800 p-1" title="Xóa">
                                <i class="fas fa-trash"></i>
                            </button>
                        </div>
                    </td>
                </tr>
            `).join('');
        }

        function loadCustomers() {
            const customerList = document.getElementById('customerList');
            
            // Get unique customers
            const allFiles = [
                ...fileDatabase.contracts,
                ...fileDatabase.sampling,
                ...fileDatabase.analysis,
                ...fileDatabase.results
            ];
            
            const customers = [...new Set(allFiles.map(f => f.customer))];
            
            if (customers.length === 0) {
                customerList.innerHTML = '<p class="text-gray-500">Chưa có khách hàng nào</p>';
                return;
            }
            
            customerList.innerHTML = customers.map(customer => {
                const customerFiles = allFiles.filter(f => f.customer === customer);
                
                return `
                    <div class="border border-gray-200 rounded-lg p-4 hover:shadow-md transition-shadow">
                        <div class="flex justify-between items-start">
                            <div>
                                <h5 class="font-semibold text-gray-800">${customer}</h5>
                                <p class="text-sm text-gray-600">Tổng: ${customerFiles.length} file</p>
                            </div>
                            <span class="bg-blue-100 text-blue-800 px-2 py-1 rounded text-sm">${customerFiles.length}</span>
                        </div>
                    </div>
                `;
            }).join('');
        }

        function loadReports() {
            document.getElementById('reportContracts').textContent = fileDatabase.contracts.length;
            document.getElementById('reportSamples').textContent = fileDatabase.sampling.length;
            document.getElementById('reportAnalysis').textContent = fileDatabase.analysis.length;
            document.getElementById('reportResults').textContent = fileDatabase.results.length;
        }

        // Customer management functions
        function loadCustomerOptions() {
            const select = document.getElementById('uploadCustomer');
            if (!select) return;
            
            // Clear existing options except the first one
            select.innerHTML = '<option value="">Chọn khách hàng</option>';
            
            // Add customers from list
            customerList.forEach(customer => {
                const option = document.createElement('option');
                option.value = customer;
                option.textContent = customer;
                select.appendChild(option);
            });
        }

        function addNewCustomer() {
            const input = document.getElementById('newCustomerInput');
            const select = document.getElementById('uploadCustomer');
            
            if (input.classList.contains('hidden')) {
                // Show input
                input.classList.remove('hidden');
                input.focus();
                
                // Handle Enter key
                input.onkeypress = function(e) {
                    if (e.key === 'Enter') {
                        saveNewCustomer();
                    }
                };
                
                // Handle blur
                input.onblur = function() {
                    if (input.value.trim()) {
                        saveNewCustomer();
                    } else {
                        input.classList.add('hidden');
                    }
                };
            }
        }

        function saveNewCustomer() {
            const input = document.getElementById('newCustomerInput');
            const customerName = input.value.trim();
            
            if (customerName && !customerList.includes(customerName)) {
                customerList.push(customerName);
                customerList.sort(); // Sort alphabetically
                saveToLocal();
                loadCustomerOptions();
                
                // Select the new customer
                document.getElementById('uploadCustomer').value = customerName;
                showNotification(`Đã thêm khách hàng: ${customerName}`, 'success');
            }
            
            input.value = '';
            input.classList.add('hidden');
        }

        // Export/Import functions
        function exportAllData() {
            const exportData = {
                version: '1.0',
                exportDate: new Date().toISOString(),
                data: fileDatabase,
                customers: customerList,
                totalFiles: Object.values(fileDatabase).reduce((sum, arr) => sum + arr.length, 0)
            };
            
            const blob = new Blob([JSON.stringify(exportData, null, 2)], { type: 'application/json' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `cem-hue-backup-${new Date().toISOString().split('T')[0]}.json`;
            a.click();
            URL.revokeObjectURL(url);
            
            showNotification('Đã xuất backup dữ liệu!', 'success');
        }

        function importData() {
            const input = document.createElement('input');
            input.type = 'file';
            input.accept = '.json';
            input.onchange = function(e) {
                const file = e.target.files[0];
                if (file) {
                    const reader = new FileReader();
                    reader.onload = function(e) {
                        try {
                            const importData = JSON.parse(e.target.result);
                            if (importData.data) {
                                fileDatabase = importData.data;
                                if (importData.customers) {
                                    customerList = importData.customers;
                                }
                                saveToLocal();
                                loadCustomerOptions();
                                loadTabData('dashboard');
                                showNotification('Đã import dữ liệu thành công!', 'success');
                            }
                        } catch (error) {
                            showNotification('File không hợp lệ!', 'error');
                        }
                    };
                    reader.readAsText(file);
                }
            };
            input.click();
        }
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9668c5dfb7c60ebe',t:'MTc1Mzc1MTgwOC4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
