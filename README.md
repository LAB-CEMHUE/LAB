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
                    <!-- Google Drive Status -->
                    <div id="driveStatus" class="hidden lg:flex items-center space-x-2">
                        <i class="fab fa-google-drive text-xl lg:text-2xl" id="driveIcon"></i>
                        <div class="hidden xl:block">
                            <div class="text-sm font-medium" id="driveText">Chưa kết nối</div>
                            <div class="text-xs text-gray-500" id="driveStorage">0 GB / 15 GB</div>
                        </div>
                        <button onclick="initializeDrive()" id="connectDriveBtn" class="bg-blue-600 hover:bg-blue-700 text-white px-2 lg:px-3 py-1 rounded text-xs lg:text-sm">
                            <i class="fab fa-google-drive lg:mr-1"></i><span class="hidden lg:inline">Kết nối</span>
                        </button>
                    </div>
                    
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
                <button onclick="showTab('contracts')" class="tab-btn px-3 lg:px-6 py-3 lg:py-4 font-semibold text-gray-600 hover:text-green-600 hover:bg-green-50 transition-colors whitespace-nowrap" data-role="admin_staff">
                    <i class="fas fa-file-contract mr-1 lg:mr-2"></i><span class="hidden sm:inline">Hợp đồng</span><span class="sm:hidden">HĐ</span>
                </button>
                <button onclick="showTab('sampling')" class="tab-btn px-3 lg:px-6 py-3 lg:py-4 font-semibold text-gray-600 hover:text-green-600 hover:bg-green-50 transition-colors whitespace-nowrap" data-role="sampling_staff">
                    <i class="fas fa-vial mr-1 lg:mr-2"></i><span class="hidden md:inline">Biên bản lấy mẫu</span><span class="md:hidden hidden sm:inline">Lấy mẫu</span><span class="sm:hidden">LM</span>
                </button>
                <button onclick="showTab('coding')" class="tab-btn px-3 lg:px-6 py-3 lg:py-4 font-semibold text-gray-600 hover:text-green-600 hover:bg-green-50 transition-colors whitespace-nowrap" data-role="sampling_staff">
                    <i class="fas fa-barcode mr-1 lg:mr-2"></i><span class="hidden sm:inline">Mã hóa</span><span class="sm:hidden">MH</span>
                </button>
                <button onclick="showTab('analysis')" class="tab-btn px-3 lg:px-6 py-3 lg:py-4 font-semibold text-gray-600 hover:text-green-600 hover:bg-green-50 transition-colors whitespace-nowrap" data-role="lab_staff">
                    <i class="fas fa-microscope mr-1 lg:mr-2"></i><span class="hidden sm:inline">Phân tích</span><span class="sm:hidden">PT</span>
                </button>
                <button onclick="showTab('results')" class="tab-btn px-3 lg:px-6 py-3 lg:py-4 font-semibold text-gray-600 hover:text-green-600 hover:bg-green-50 transition-colors whitespace-nowrap" data-role="lab_manager">
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
                            <p class="text-2xl lg:text-3xl font-bold" id="activeContracts">24</p>
                        </div>
                        <i class="fas fa-file-contract text-2xl lg:text-4xl text-blue-200 self-end lg:self-auto"></i>
                    </div>
                </div>
                <div class="bg-gradient-to-r from-green-500 to-green-600 text-white p-3 lg:p-6 rounded-xl shadow-lg hover-scale">
                    <div class="flex flex-col lg:flex-row lg:items-center lg:justify-between">
                        <div class="mb-2 lg:mb-0">
                            <p class="text-green-100 text-xs lg:text-sm">Mẫu nhận hôm nay</p>
                            <p class="text-2xl lg:text-3xl font-bold" id="todaySamples">12</p>
                            <p class="text-xs text-green-100 hidden lg:block">Tổng mẫu đã tiếp nhận</p>
                        </div>
                        <i class="fas fa-vial text-2xl lg:text-4xl text-green-200 self-end lg:self-auto"></i>
                    </div>
                </div>
                <div class="bg-gradient-to-r from-orange-500 to-orange-600 text-white p-3 lg:p-6 rounded-xl shadow-lg hover-scale">
                    <div class="flex flex-col lg:flex-row lg:items-center lg:justify-between">
                        <div class="mb-2 lg:mb-0">
                            <p class="text-orange-100 text-xs lg:text-sm">Đang phân tích</p>
                            <p class="text-2xl lg:text-3xl font-bold" id="inAnalysis">89</p>
                        </div>
                        <i class="fas fa-microscope text-2xl lg:text-4xl text-orange-200 self-end lg:self-auto"></i>
                    </div>
                </div>
                <div class="bg-gradient-to-r from-purple-500 to-purple-600 text-white p-3 lg:p-6 rounded-xl shadow-lg hover-scale">
                    <div class="flex flex-col lg:flex-row lg:items-center lg:justify-between">
                        <div class="mb-2 lg:mb-0">
                            <p class="text-purple-100 text-xs lg:text-sm">Dung lượng Drive</p>
                            <p class="text-xl lg:text-2xl font-bold" id="driveUsage">2.4 GB</p>
                            <p class="text-xs text-purple-100">/ 15 GB</p>
                        </div>
                        <i class="fab fa-google-drive text-2xl lg:text-4xl text-purple-200 self-end lg:self-auto"></i>
                    </div>
                </div>
            </div>

            <!-- Sample Connections Overview -->
            <div class="bg-white rounded-xl shadow-lg p-6 mb-8">
                <h3 class="text-xl font-bold text-gray-800 mb-4">
                    <i class="fas fa-project-diagram mr-2 text-green-600"></i>Mối liên hệ mẫu hôm nay
                </h3>
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4" id="sampleConnections">
                    <!-- Sample connections will be populated here -->
                </div>
            </div>

            <!-- Recent Activities -->
            <div class="bg-white rounded-xl shadow-lg p-6">
                <h3 class="text-xl font-bold text-gray-800 mb-4">
                    <i class="fas fa-clock mr-2 text-green-600"></i>Hoạt động gần đây
                </h3>
                <div class="space-y-4" id="recentActivities">
                    <!-- Activities will be populated here -->
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
                    <div class="flex space-x-2">
                        <button onclick="openUploadModal('contract')" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg transition-colors">
                            <i class="fas fa-upload mr-2"></i>Tải lên hợp đồng
                        </button>
                        <button onclick="exportCustomerFiles('contracts')" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg transition-colors">
                            <i class="fas fa-file-export mr-2"></i>Xuất hồ sơ
                        </button>
                    </div>
                </div>
                
                <div class="overflow-x-auto">
                    <table class="w-full table-auto min-w-full">
                        <thead class="bg-gray-50">
                            <tr>
                                <th class="px-2 lg:px-4 py-3 text-left font-semibold text-gray-700 text-sm">Mã HĐ</th>
                                <th class="px-2 lg:px-4 py-3 text-left font-semibold text-gray-700 text-sm">Khách hàng</th>
                                <th class="px-2 lg:px-4 py-3 text-left font-semibold text-gray-700 text-sm hidden md:table-cell">Ngày ký</th>
                                <th class="px-2 lg:px-4 py-3 text-left font-semibold text-gray-700 text-sm">File</th>
                                <th class="px-2 lg:px-4 py-3 text-left font-semibold text-gray-700 text-sm hidden lg:table-cell">Trạng thái</th>
                                <th class="px-2 lg:px-4 py-3 text-left font-semibold text-gray-700 text-sm">Thao tác</th>
                            </tr>
                        </thead>
                        <tbody class="divide-y divide-gray-200" id="contractsTable">
                            <!-- Contract data will be populated here -->
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
                    <div class="flex space-x-2">
                        <button onclick="openUploadModal('sampling')" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg transition-colors">
                            <i class="fas fa-upload mr-2"></i>Tải lên biên bản
                        </button>
                        <button onclick="showSampleConnections()" class="bg-purple-600 hover:bg-purple-700 text-white px-4 py-2 rounded-lg transition-colors">
                            <i class="fas fa-project-diagram mr-2"></i>Xem mối liên hệ
                        </button>
                    </div>
                </div>
                
                <div class="overflow-x-auto">
                    <table class="w-full table-auto">
                        <thead class="bg-gray-50">
                            <tr>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Mã BB</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Khách hàng</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Ngày lấy mẫu</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Loại mẫu</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Số lượng</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">File đính kèm</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Mối liên hệ</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Thao tác</th>
                            </tr>
                        </thead>
                        <tbody class="divide-y divide-gray-200" id="samplingTable">
                            <!-- Sampling data will be populated here -->
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
                    <!-- Coding data will be populated here -->
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
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Mã mẫu</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Ngày phân tích</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Người thực hiện</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">File đính kèm</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Trạng thái</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Thao tác</th>
                            </tr>
                        </thead>
                        <tbody class="divide-y divide-gray-200" id="analysisTable">
                            <!-- Analysis data will be populated here -->
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
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Mã mẫu</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Khách hàng</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Ngày hoàn thành</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">File đính kèm</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Kết quả</th>
                                <th class="px-4 py-3 text-left font-semibold text-gray-700">Thao tác</th>
                            </tr>
                        </thead>
                        <tbody class="divide-y divide-gray-200" id="resultsTable">
                            <!-- Results data will be populated here -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- Reports Tab -->
        <div id="reports" class="tab-content hidden">
            <div class="space-y-6">
                <!-- Report Filters -->
                <div class="bg-white rounded-xl shadow-lg p-4 lg:p-6">
                    <h3 class="text-lg lg:text-xl font-bold text-gray-800 mb-4">
                        <i class="fas fa-filter mr-2 text-blue-600"></i>Bộ lọc báo cáo
                    </h3>
                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Thời gian</label>
                            <select id="reportPeriod" class="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm" onchange="updateReports()">
                                <option value="today">Hôm nay</option>
                                <option value="week">Tuần này</option>
                                <option value="month" selected>Tháng này</option>
                                <option value="quarter">Quý này</option>
                                <option value="year">Năm này</option>
                                <option value="custom">Tùy chọn</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Khách hàng</label>
                            <select id="reportCustomer" class="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm" onchange="updateReports()">
                                <option value="">Tất cả khách hàng</option>
                                <option value="cty-abc">Công ty ABC</option>
                                <option value="cty-xyz">Công ty XYZ</option>
                                <option value="cty-def">Công ty DEF</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Loại báo cáo</label>
                            <select id="reportType" class="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm" onchange="updateReports()">
                                <option value="samples">Theo mẫu</option>
                                <option value="dtm">ĐTM (Đánh giá tác động môi trường)</option>
                                <option value="gpmt">GPMT (Giấy phép môi trường)</option>
                                <option value="monitoring">Báo cáo giám sát</option>
                                <option value="environmental_status">Báo cáo hiện trạng môi trường</option>
                            </select>
                        </div>
                        <div class="flex items-end">
                            <button onclick="exportReport()" class="w-full bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg text-sm">
                                <i class="fas fa-file-export mr-2"></i>Xuất báo cáo
                            </button>
                        </div>
                    </div>
                </div>

                <!-- Statistics Cards -->
                <div class="grid grid-cols-2 lg:grid-cols-4 gap-3 lg:gap-6">
                    <div class="bg-white rounded-xl shadow-lg p-4 lg:p-6">
                        <div class="flex items-center justify-between">
                            <div>
                                <p class="text-gray-600 text-sm">Tổng mẫu tháng</p>
                                <p class="text-2xl font-bold text-blue-600" id="monthlyTotal">156</p>
                                <p class="text-xs text-green-600">+12% so với tháng trước</p>
                            </div>
                            <i class="fas fa-vial text-3xl text-blue-200"></i>
                        </div>
                    </div>
                    <div class="bg-white rounded-xl shadow-lg p-4 lg:p-6">
                        <div class="flex items-center justify-between">
                            <div>
                                <p class="text-gray-600 text-sm">Trung bình/ngày</p>
                                <p class="text-2xl font-bold text-green-600" id="dailyAverage">5.2</p>
                                <p class="text-xs text-blue-600">Ổn định</p>
                            </div>
                            <i class="fas fa-chart-line text-3xl text-green-200"></i>
                        </div>
                    </div>
                    <div class="bg-white rounded-xl shadow-lg p-4 lg:p-6">
                        <div class="flex items-center justify-between">
                            <div>
                                <p class="text-gray-600 text-sm">Tỷ lệ đạt chuẩn</p>
                                <p class="text-2xl font-bold text-purple-600" id="passRate">94.2%</p>
                                <p class="text-xs text-green-600">+2.1% so với tháng trước</p>
                            </div>
                            <i class="fas fa-check-circle text-3xl text-purple-200"></i>
                        </div>
                    </div>
                    <div class="bg-white rounded-xl shadow-lg p-4 lg:p-6">
                        <div class="flex items-center justify-between">
                            <div>
                                <p class="text-gray-600 text-sm">Thời gian TB</p>
                                <p class="text-2xl font-bold text-orange-600" id="avgTime">3.2</p>
                                <p class="text-xs text-gray-600">ngày/mẫu</p>
                            </div>
                            <i class="fas fa-clock text-3xl text-orange-200"></i>
                        </div>
                    </div>
                </div>

                <!-- Charts Section -->
                <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                    <!-- Sample Trend Chart -->
                    <div class="bg-white rounded-xl shadow-lg p-4 lg:p-6">
                        <h4 class="text-lg font-bold text-gray-800 mb-4">
                            <i class="fas fa-chart-area mr-2 text-blue-600"></i>Xu hướng lấy mẫu
                        </h4>
                        <div class="h-64">
                            <canvas id="sampleTrendChart"></canvas>
                        </div>
                    </div>

                    <!-- Customer Distribution -->
                    <div class="bg-white rounded-xl shadow-lg p-4 lg:p-6">
                        <h4 class="text-lg font-bold text-gray-800 mb-4">
                            <i class="fas fa-chart-pie mr-2 text-green-600"></i>Phân bố khách hàng
                        </h4>
                        <div class="h-64">
                            <canvas id="customerChart"></canvas>
                        </div>
                    </div>
                </div>

                <!-- Performance Table -->
                <div class="bg-white rounded-xl shadow-lg p-4 lg:p-6">
                    <div class="flex flex-col lg:flex-row lg:justify-between lg:items-center mb-4">
                        <h4 class="text-lg font-bold text-gray-800 mb-2 lg:mb-0">
                            <i class="fas fa-table mr-2 text-purple-600"></i>Bảng hiệu suất chi tiết
                        </h4>
                        <div class="flex space-x-2">
                            <button onclick="refreshReportData()" class="bg-blue-600 hover:bg-blue-700 text-white px-3 py-2 rounded-lg text-sm">
                                <i class="fas fa-sync-alt mr-1"></i>Làm mới
                            </button>
                            <button onclick="exportTableData()" class="bg-green-600 hover:bg-green-700 text-white px-3 py-2 rounded-lg text-sm">
                                <i class="fas fa-download mr-1"></i>Tải xuống
                            </button>
                        </div>
                    </div>
                    <div class="overflow-x-auto">
                        <table class="w-full table-auto">
                            <thead class="bg-gray-50">
                                <tr>
                                    <th class="px-4 py-3 text-left font-semibold text-gray-700">Khách hàng</th>
                                    <th class="px-4 py-3 text-left font-semibold text-gray-700">Tổng mẫu</th>
                                    <th class="px-4 py-3 text-left font-semibold text-gray-700">Đã hoàn thành</th>
                                    <th class="px-4 py-3 text-left font-semibold text-gray-700">Đang xử lý</th>
                                    <th class="px-4 py-3 text-left font-semibold text-gray-700">Tỷ lệ đạt</th>
                                    <th class="px-4 py-3 text-left font-semibold text-gray-700">Thời gian TB</th>
                                </tr>
                            </thead>
                            <tbody class="divide-y divide-gray-200" id="performanceTable">
                                <!-- Performance data will be populated here -->
                            </tbody>
                        </table>
                    </div>
                </div>

                <!-- Quality Analysis -->
                <div class="bg-white rounded-xl shadow-lg p-4 lg:p-6">
                    <h4 class="text-lg font-bold text-gray-800 mb-4">
                        <i class="fas fa-microscope mr-2 text-orange-600"></i>Phân tích chất lượng
                    </h4>
                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                        <div class="bg-green-50 p-4 rounded-lg">
                            <div class="flex items-center justify-between mb-2">
                                <h5 class="font-semibold text-green-800">Đạt chuẩn QCVN</h5>
                                <i class="fas fa-check-circle text-green-600"></i>
                            </div>
                            <p class="text-2xl font-bold text-green-600">147 mẫu</p>
                            <p class="text-sm text-green-700">94.2% tổng số mẫu</p>
                        </div>
                        <div class="bg-yellow-50 p-4 rounded-lg">
                            <div class="flex items-center justify-between mb-2">
                                <h5 class="font-semibold text-yellow-800">Cần theo dõi</h5>
                                <i class="fas fa-exclamation-triangle text-yellow-600"></i>
                            </div>
                            <p class="text-2xl font-bold text-yellow-600">7 mẫu</p>
                            <p class="text-sm text-yellow-700">4.5% tổng số mẫu</p>
                        </div>
                        <div class="bg-red-50 p-4 rounded-lg">
                            <div class="flex items-center justify-between mb-2">
                                <h5 class="font-semibold text-red-800">Vượt ngưỡng</h5>
                                <i class="fas fa-times-circle text-red-600"></i>
                            </div>
                            <p class="text-2xl font-bold text-red-600">2 mẫu</p>
                            <p class="text-sm text-red-700">1.3% tổng số mẫu</p>
                        </div>
                    </div>
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
                    <div class="flex space-x-2">
                        <select id="customerFilter" class="border border-gray-300 rounded-lg px-3 py-2">
                            <option value="">Tất cả khách hàng</option>
                        </select>
                        <button onclick="exportCustomerProfile()" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg transition-colors">
                            <i class="fas fa-file-export mr-2"></i>Xuất hồ sơ khách hàng
                        </button>
                    </div>
                </div>
                
                <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                    <!-- Customer List -->
                    <div>
                        <h4 class="font-semibold text-gray-700 mb-4">Danh sách khách hàng</h4>
                        <div class="space-y-3" id="customerList">
                            <!-- Customer list will be populated here -->
                        </div>
                    </div>
                    
                    <!-- Customer Progress -->
                    <div>
                        <h4 class="font-semibold text-gray-700 mb-4">Tiến độ theo dõi</h4>
                        <div id="customerProgress" class="bg-gray-50 rounded-lg p-4 text-center text-gray-500">
                            Chọn khách hàng để xem tiến độ
                        </div>
                    </div>
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
                        <select id="uploadCustomer" class="w-full border border-gray-300 rounded-lg px-3 py-2" required>
                            <option value="">Chọn khách hàng</option>
                            <option value="cty-abc">Công ty ABC</option>
                            <option value="cty-xyz">Công ty XYZ</option>
                            <option value="cty-def">Công ty DEF</option>
                        </select>
                    </div>
                    
                    <div class="mb-4">
                        <label class="block text-sm font-medium text-gray-700 mb-2">Mã tham chiếu</label>
                        <input type="text" id="uploadReference" class="w-full border border-gray-300 rounded-lg px-3 py-2" placeholder="Ví dụ: HD001, BB001, MA001..." required>
                    </div>
                    
                    <div class="mb-4" id="sampleConnectionDiv" style="display: none;">
                        <label class="block text-sm font-medium text-gray-700 mb-2">Mối liên hệ với mẫu khác</label>
                        <input type="text" id="sampleConnections" class="w-full border border-gray-300 rounded-lg px-3 py-2" placeholder="Ví dụ: BB002, BB003 (cách nhau bằng dấu phẩy)">
                        <p class="text-xs text-gray-500 mt-1">Nhập mã các mẫu có liên quan</p>
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
                    <button onclick="shareFile(currentPreviewFile)" class="bg-blue-600 hover:bg-blue-700 text-white px-3 py-2 rounded-lg text-sm">
                        <i class="fas fa-share-alt mr-1"></i>Chia sẻ
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

    <!-- Sample Connections Modal -->
    <div id="connectionsModal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50">
        <div class="bg-white rounded-xl w-full max-w-4xl mx-4 h-4/5 flex flex-col">
            <div class="flex justify-between items-center p-6 border-b">
                <h3 class="text-lg font-bold text-purple-600">
                    <i class="fas fa-project-diagram mr-2"></i>Mối liên hệ giữa các mẫu
                </h3>
                <button onclick="closeModal('connectionsModal')" class="text-gray-500 hover:text-gray-700">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            <div class="flex-1 p-6 overflow-auto">
                <div id="connectionsContent">
                    <!-- Connections visualization will be here -->
                </div>
            </div>
        </div>
    </div>

    <!-- QR Code Modal -->
    <div id="qrModal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50">
        <div class="bg-white rounded-xl w-full max-w-md mx-4">
            <div class="flex justify-between items-center p-6 border-b">
                <h3 class="text-lg font-bold text-blue-600">
                    <i class="fas fa-qrcode mr-2"></i>QR Code Mẫu
                </h3>
                <button onclick="closeModal('qrModal')" class="text-gray-500 hover:text-gray-700">
                    <i class="fas fa-times text-xl"></i>
                </button>
            </div>
            <div class="p-6 text-center">
                <div id="qrCodeContainer" class="mb-4">
                    <!-- QR code will be generated here -->
                </div>
                <p id="qrCodeText" class="text-sm text-gray-600 mb-4">Mã QR cho mẫu</p>
                <button onclick="downloadQRCode()" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg mr-2">
                    <i class="fas fa-download mr-2"></i>Tải xuống
                </button>
                <button onclick="printQRCode()" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg">
                    <i class="fas fa-print mr-2"></i>In QR Code
                </button>
            </div>
        </div>
    </div>

    <script>
        // 🆓 PHIÊN BẢN MIỄN PHÍ - Lưu trữ local
        let currentPreviewFile = null;
        let currentUploadType = '';
        let fileDatabase = {
            contracts: [],
            sampling: [],
            coding: [],
            analysis: [],
            results: []
        };

        // Lưu dữ liệu vào localStorage (miễn phí)
        function saveToLocal() {
            localStorage.setItem('cemHueLabData', JSON.stringify(fileDatabase));
            localStorage.setItem('cemHueLabLastUpdate', new Date().toISOString());
        }

        // Đọc dữ liệu từ localStorage
        function loadFromLocal() {
            const saved = localStorage.getItem('cemHueLabData');
            if (saved) {
                fileDatabase = JSON.parse(saved);
                console.log('✅ Đã tải dữ liệu từ bộ nhớ local');
            }
        }

        // Export dữ liệu ra file JSON (backup miễn phí)
        function exportAllData() {
            const exportData = {
                version: '1.0',
                exportDate: new Date().toISOString(),
                data: fileDatabase,
                totalFiles: Object.values(fileDatabase).reduce((sum, arr) => sum + arr.length, 0)
            };
            
            const blob = new Blob([JSON.stringify(exportData, null, 2)], { type: 'application/json' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `cem-hue-backup-${new Date().toISOString().split('T')[0]}.json`;
            a.click();
            URL.revokeObjectURL(url);
            
            showNotification('💾 Đã xuất backup dữ liệu!');
        }

        // Import dữ liệu từ file JSON
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
                                saveToLocal();
                                loadTabData(getCurrentTab());
                                showNotification('📥 Đã import dữ liệu thành công!');
                            }
                        } catch (error) {
                            showNotification('❌ File không hợp lệ!');
                        }
                    };
                    reader.readAsText(file);
                }
            };
            input.click();
        }

        // Google Drive Configuration (Simple API Key approach)
        const GOOGLE_API_KEY = 'your-api-key-here'; // Replace with your API key
        const DISCOVERY_DOC = 'https://www.googleapis.com/discovery/v1/apis/drive/v3/rest';

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

        // Initialize Google Drive
        async function initializeDrive() {
            if (GOOGLE_API_KEY === 'YOUR_API_KEY_HERE') {
                showDriveSetupGuide();
                return;
            }

            try {
                await gapi.load('client', async () => {
                    await gapi.client.init({
                        apiKey: GOOGLE_API_KEY,
                        discoveryDocs: [DISCOVERY_DOC],
                    });
                    
                    driveInitialized = true;
                    updateDriveStatus(true);
                    showNotification('✅ Đã kết nối Google Drive thành công!');
                    
                    // Get drive usage
                    getDriveUsage();
                });
            } catch (error) {
                console.error('Drive initialization error:', error);
                showNotification('❌ Lỗi kết nối Google Drive. Vui lòng kiểm tra API Key.');
            }
        }

        function showDriveSetupGuide() {
            const modal = document.createElement('div');
            modal.className = 'fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50';
            modal.innerHTML = `
                <div class="bg-white rounded-xl p-6 w-full max-w-2xl mx-4">
                    <div class="flex justify-between items-center mb-4">
                        <h3 class="text-lg font-bold text-blue-600">
                            <i class="fab fa-google-drive mr-2"></i>Hướng dẫn kết nối Google Drive
                        </h3>
                        <button onclick="this.closest('.fixed').remove()" class="text-gray-500 hover:text-gray-700">
                            <i class="fas fa-times"></i>
                        </button>
                    </div>
                    
                    <div class="space-y-4">
                        <div class="bg-blue-50 p-4 rounded-lg">
                            <h4 class="font-semibold text-blue-800 mb-2">Bước 1: Tạo API Key</h4>
                            <ol class="list-decimal list-inside space-y-1 text-sm text-blue-700">
                                <li>Vào <a href="https://console.cloud.google.com/apis/credentials" target="_blank" class="underline">Google Cloud Console</a></li>
                                <li>Tạo project mới hoặc chọn project có sẵn</li>
                                <li>Click "Create Credentials" → "API Key"</li>
                                <li>Copy API Key vừa tạo</li>
                            </ol>
                        </div>
                        
                        <div class="bg-green-50 p-4 rounded-lg">
                            <h4 class="font-semibold text-green-800 mb-2">Bước 2: Bật Google Drive API</h4>
                            <ol class="list-decimal list-inside space-y-1 text-sm text-green-700">
                                <li>Vào "APIs & Services" → "Library"</li>
                                <li>Tìm "Google Drive API" → Click "Enable"</li>
                            </ol>
                        </div>
                        
                        <div class="bg-yellow-50 p-4 rounded-lg">
                            <h4 class="font-semibold text-yellow-800 mb-2">Bước 3: Cập nhật code</h4>
                            <p class="text-sm text-yellow-700 mb-2">Thay thế dòng này trong code:</p>
                            <code class="block bg-gray-100 p-2 rounded text-sm">
                                const GOOGLE_API_KEY = 'your-api-key-here';
                            </code>
                        </div>
                        
                        <div class="flex justify-center space-x-4">
                            <button onclick="window.open('https://console.cloud.google.com/apis/credentials', '_blank')" 
                                    class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg">
                                <i class="fab fa-google mr-2"></i>Mở Google Console
                            </button>
                            <button onclick="this.closest('.fixed').remove()" 
                                    class="bg-gray-500 hover:bg-gray-600 text-white px-4 py-2 rounded-lg">
                                Đóng
                            </button>
                        </div>
                    </div>
                </div>
            `;
            document.body.appendChild(modal);
        }

        async function getDriveUsage() {
            if (!driveInitialized) return;
            
            try {
                const response = await gapi.client.drive.about.get({
                    fields: 'storageQuota'
                });
                
                const quota = response.result.storageQuota;
                const used = parseInt(quota.usage) / (1024 * 1024 * 1024); // Convert to GB
                const total = parseInt(quota.limit) / (1024 * 1024 * 1024); // Convert to GB
                
                document.getElementById('driveStorage').textContent = `${used.toFixed(1)} GB / ${total.toFixed(0)} GB`;
                document.getElementById('driveUsage').textContent = `${used.toFixed(1)} GB`;
            } catch (error) {
                console.error('Error getting drive usage:', error);
            }
        }

        function updateDriveStatus(connected) {
            const icon = document.getElementById('driveIcon');
            const text = document.getElementById('driveText');
            const btn = document.getElementById('connectDriveBtn');
            
            if (connected) {
                icon.className = 'fab fa-google-drive text-2xl text-green-600';
                text.textContent = 'Đã kết nối';
                text.className = 'text-sm font-medium text-green-600';
                btn.style.display = 'none';
            } else {
                icon.className = 'fab fa-google-drive text-2xl text-gray-400';
                text.textContent = 'Chưa kết nối';
                text.className = 'text-sm font-medium text-gray-500';
                btn.style.display = 'inline-block';
            }
        }

        // Tab switching
        function showTab(tabName) {
            const userRole = document.getElementById('userRole').value;
            if (!hasPermission(tabName, userRole)) {
                showNotification('⚠️ Bạn không có quyền truy cập mục này!');
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
                activeBtn.classList.add('active', 'text-blue-600', 'border-b-2', 'border-blue-600', 'bg-blue-50');
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
            showNotification(`Đã chuyển sang vai trò: ${roleNames[role]}`);
        }

        // Upload functionality
        function openUploadModal(type) {
            currentUploadType = type;
            const titles = {
                contract: 'Tải lên Hợp đồng',
                sampling: 'Tải lên Biên bản Lấy mẫu',
                coding: 'Tải lên Danh mục Mã hóa',
                analysis: 'Tải lên Biên bản Phân tích',
                results: 'Tải lên Kết quả Thử nghiệm'
            };
            
            document.getElementById('uploadModalTitle').textContent = titles[type];
            
            // Show sample connection field for sampling
            const connectionDiv = document.getElementById('sampleConnectionDiv');
            if (type === 'sampling') {
                connectionDiv.style.display = 'block';
            } else {
                connectionDiv.style.display = 'none';
            }
            
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
            }
        }

        // Drag and drop functionality
        document.addEventListener('DOMContentLoaded', function() {
            const dropZone = document.getElementById('dropZone');
            const fileInput = document.getElementById('fileInput');
            
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
        });

        function handleFiles(files) {
            const fileList = document.getElementById('fileList');
            fileList.innerHTML = '';
            
            Array.from(files).forEach((file, index) => {
                if (file.size > 10 * 1024 * 1024) { // 10MB limit
                    showNotification(`File ${file.name} quá lớn (>10MB)`);
                    return;
                }
                
                const fileItem = createFileItem(file, index);
                fileList.appendChild(fileItem);
            });
        }

        function createFileItem(file, index) {
            const div = document.createElement('div');
            div.className = 'flex items-center justify-between p-3 bg-gray-50 rounded-lg';
            
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

        // Upload form submission
        document.getElementById('uploadForm').addEventListener('submit', async function(e) {
            e.preventDefault();
            
            const customer = document.getElementById('uploadCustomer').value;
            const reference = document.getElementById('uploadReference').value;
            const connections = document.getElementById('sampleConnections').value;
            const files = document.getElementById('fileInput').files;
            
            if (!customer || !reference || files.length === 0) {
                showNotification('⚠️ Vui lòng điền đầy đủ thông tin!');
                return;
            }
            
            // Simulate upload process
            await simulateUpload(files, customer, reference, connections);
            
            closeModal('uploadModal');
            loadTabData(currentUploadType);
            showNotification('✅ Tải lên thành công!');
        });

        async function simulateUpload(files, customer, reference, connections) {
            const progressBars = document.querySelectorAll('.progress-bar');
            
            for (let i = 0; i < files.length; i++) {
                const file = files[i];
                const progressBar = progressBars[i];
                
                // Simulate upload progress
                for (let progress = 0; progress <= 100; progress += 10) {
                    progressBar.style.width = progress + '%';
                    await new Promise(resolve => setTimeout(resolve, 100));
                }
                
                // Convert file to base64 for local storage (miễn phí)
                const base64Data = await fileToBase64(file);
                
                // Add to database
                const fileData = {
                    id: Date.now() + i,
                    name: file.name,
                    size: file.size,
                    type: getFileType(file.name),
                    customer: customer,
                    reference: reference,
                    connections: connections ? connections.split(',').map(s => s.trim()) : [],
                    uploadDate: new Date().toISOString(),
                    localData: base64Data, // Lưu trữ local thay vì Drive
                    driveId: 'local_' + Date.now() + i
                };
                
                fileDatabase[currentUploadType].push(fileData);
            }
            
            // Lưu vào localStorage sau khi upload
            saveToLocal();
        }

        // Chuyển file thành base64 để lưu local
        function fileToBase64(file) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.readAsDataURL(file);
                reader.onload = () => resolve(reader.result);
                reader.onerror = error => reject(error);
            });
        }

        // File preview
        async function previewFile(fileData) {
            currentPreviewFile = fileData;
            
            document.getElementById('previewTitle').textContent = fileData.name;
            document.getElementById('previewSubtitle').textContent = `${fileData.customer} - ${formatFileSize(fileData.size)}`;
            
            const previewContent = document.getElementById('previewContent');
            
            // Generate preview based on file type
            switch (fileData.type) {
                case 'word':
                    previewContent.innerHTML = generateWordPreview(fileData);
                    break;
                case 'excel':
                    previewContent.innerHTML = generateExcelPreview(fileData);
                    break;
                case 'pdf':
                    previewContent.innerHTML = generatePdfPreview(fileData);
                    break;
                case 'image':
                    previewContent.innerHTML = generateImagePreview(fileData);
                    break;
                default:
                    previewContent.innerHTML = generateGenericPreview(fileData);
            }
            
            document.getElementById('previewModal').classList.remove('hidden');
            document.getElementById('previewModal').classList.add('flex');
        }

        function generateWordPreview(fileData) {
            return `
                <div class="bg-white border rounded-lg p-6 shadow-sm">
                    <div class="flex items-center mb-4">
                        <i class="fas fa-file-word text-blue-600 text-2xl mr-3"></i>
                        <div>
                            <h4 class="font-bold text-gray-800">Tài liệu Word</h4>
                            <p class="text-sm text-gray-600">${fileData.name}</p>
                        </div>
                    </div>
                    <div class="bg-gray-50 p-4 rounded border-l-4 border-blue-500">
                        <h5 class="font-semibold mb-2">Nội dung mẫu:</h5>
                        <p class="text-gray-700 leading-relaxed">
                            <strong>BIÊN BẢN LẤY MẪU</strong><br><br>
                            Khách hàng: ${fileData.customer}<br>
                            Mã tham chiếu: ${fileData.reference}<br>
                            Ngày lấy mẫu: ${new Date().toLocaleDateString('vi-VN')}<br><br>
                            1. Thông tin mẫu<br>
                            - Loại mẫu: Nước thải<br>
                            - Số lượng: 5 mẫu<br>
                            - Vị trí lấy mẫu: Đầu ra xử lý<br><br>
                            2. Điều kiện lấy mẫu<br>
                            - Thời tiết: Nắng, nhiệt độ 28°C<br>
                            - pH hiện trường: 7.2<br>
                            - Nhiệt độ mẫu: 26°C<br><br>
                            <em>Đây là nội dung mẫu để demo preview...</em>
                        </p>
                    </div>
                </div>
            `;
        }

        function generateExcelPreview(fileData) {
            return `
                <div class="bg-white border rounded-lg p-6 shadow-sm">
                    <div class="flex items-center mb-4">
                        <i class="fas fa-file-excel text-green-600 text-2xl mr-3"></i>
                        <div>
                            <h4 class="font-bold text-gray-800">Bảng tính Excel</h4>
                            <p class="text-sm text-gray-600">${fileData.name}</p>
                        </div>
                    </div>
                    <div class="overflow-x-auto">
                        <table class="w-full border-collapse border border-gray-300">
                            <thead class="bg-green-50">
                                <tr>
                                    <th class="border border-gray-300 px-4 py-2 text-left">STT</th>
                                    <th class="border border-gray-300 px-4 py-2 text-left">Mã mẫu</th>
                                    <th class="border border-gray-300 px-4 py-2 text-left">Loại mẫu</th>
                                    <th class="border border-gray-300 px-4 py-2 text-left">pH</th>
                                    <th class="border border-gray-300 px-4 py-2 text-left">COD (mg/L)</th>
                                    <th class="border border-gray-300 px-4 py-2 text-left">TSS (mg/L)</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr>
                                    <td class="border border-gray-300 px-4 py-2">1</td>
                                    <td class="border border-gray-300 px-4 py-2">${fileData.reference}-01</td>
                                    <td class="border border-gray-300 px-4 py-2">Nước thải</td>
                                    <td class="border border-gray-300 px-4 py-2">7.2</td>
                                    <td class="border border-gray-300 px-4 py-2">45</td>
                                    <td class="border border-gray-300 px-4 py-2">32</td>
                                </tr>
                                <tr>
                                    <td class="border border-gray-300 px-4 py-2">2</td>
                                    <td class="border border-gray-300 px-4 py-2">${fileData.reference}-02</td>
                                    <td class="border border-gray-300 px-4 py-2">Nước thải</td>
                                    <td class="border border-gray-300 px-4 py-2">7.1</td>
                                    <td class="border border-gray-300 px-4 py-2">42</td>
                                    <td class="border border-gray-300 px-4 py-2">28</td>
                                </tr>
                                <tr>
                                    <td class="border border-gray-300 px-4 py-2">3</td>
                                    <td class="border border-gray-300 px-4 py-2">${fileData.reference}-03</td>
                                    <td class="border border-gray-300 px-4 py-2">Nước thải</td>
                                    <td class="border border-gray-300 px-4 py-2">7.3</td>
                                    <td class="border border-gray-300 px-4 py-2">48</td>
                                    <td class="border border-gray-300 px-4 py-2">35</td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                    <p class="text-sm text-gray-500 mt-4"><em>Đây là dữ liệu mẫu để demo preview...</em></p>
                </div>
            `;
        }

        function generatePdfPreview(fileData) {
            return `
                <div class="bg-white border rounded-lg p-6 shadow-sm">
                    <div class="flex items-center mb-4">
                        <i class="fas fa-file-pdf text-red-600 text-2xl mr-3"></i>
                        <div>
                            <h4 class="font-bold text-gray-800">Tài liệu PDF</h4>
                            <p class="text-sm text-gray-600">${fileData.name}</p>
                        </div>
                    </div>
                    <div class="bg-gray-100 p-8 rounded-lg text-center">
                        <i class="fas fa-file-pdf text-6xl text-red-400 mb-4"></i>
                        <p class="text-gray-600 mb-4">Preview PDF: ${fileData.name}</p>
                        <div class="bg-white p-4 rounded shadow-sm text-left max-w-md mx-auto">
                            <h5 class="font-bold mb-2">PHIẾU KẾT QUẢ THỬ NGHIỆM</h5>
                            <hr class="mb-2">
                            <p><strong>Khách hàng:</strong> ${fileData.customer}</p>
                            <p><strong>Mã phiếu:</strong> ${fileData.reference}</p>
                            <p><strong>Ngày:</strong> ${new Date().toLocaleDateString('vi-VN')}</p>
                            <br>
                            <p><strong>Kết quả phân tích:</strong></p>
                            <p>• pH: 7.2 (Đạt QCVN)</p>
                            <p>• COD: 45 mg/L (Đạt QCVN)</p>
                            <p>• TSS: 32 mg/L (Đạt QCVN)</p>
                            <br>
                            <p class="text-sm text-gray-500"><em>Nội dung mẫu...</em></p>
                        </div>
                    </div>
                </div>
            `;
        }

        function generateImagePreview(fileData) {
            return `
                <div class="bg-white border rounded-lg p-6 shadow-sm">
                    <div class="flex items-center mb-4">
                        <i class="fas fa-file-image text-purple-600 text-2xl mr-3"></i>
                        <div>
                            <h4 class="font-bold text-gray-800">Hình ảnh</h4>
                            <p class="text-sm text-gray-600">${fileData.name}</p>
                        </div>
                    </div>
                    <div class="bg-gray-100 p-8 rounded-lg text-center">
                        <i class="fas fa-image text-6xl text-purple-400 mb-4"></i>
                        <p class="text-gray-600 mb-4">Hình ảnh hiện trường lấy mẫu</p>
                        <div class="bg-gradient-to-br from-blue-100 to-green-100 p-8 rounded-lg max-w-md mx-auto">
                            <p class="text-gray-700">📸 Ảnh chụp hiện trường</p>
                            <p class="text-sm text-gray-600 mt-2">Vị trí: Đầu ra xử lý nước thải</p>
                            <p class="text-sm text-gray-600">Thời gian: ${new Date().toLocaleString('vi-VN')}</p>
                            <p class="text-sm text-gray-600">Kích thước: ${formatFileSize(fileData.size)}</p>
                        </div>
                        <p class="text-sm text-gray-500 mt-4"><em>Đây là preview mẫu cho hình ảnh...</em></p>
                    </div>
                </div>
            `;
        }

        function generateGenericPreview(fileData) {
            return `
                <div class="bg-white border rounded-lg p-6 shadow-sm text-center">
                    <i class="fas fa-file text-6xl text-gray-400 mb-4"></i>
                    <h4 class="font-bold text-gray-800 mb-2">${fileData.name}</h4>
                    <p class="text-gray-600 mb-4">Không thể preview loại file này</p>
                    <button onclick="downloadFile(currentPreviewFile)" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg">
                        <i class="fas fa-download mr-2"></i>Tải xuống để xem
                    </button>
                </div>
            `;
        }

        // File operations
        function downloadFile(fileData) {
            if (fileData.localData) {
                // Download từ local storage (miễn phí)
                const link = document.createElement('a');
                link.href = fileData.localData;
                link.download = fileData.name;
                link.click();
                showNotification(`📥 Đã tải xuống ${fileData.name}`);
            } else {
                showNotification(`⚠️ File ${fileData.name} không có dữ liệu local`);
            }
        }

        function shareFile(fileData) {
            const shareUrl = `https://drive.google.com/file/d/${fileData.driveId}/view`;
            navigator.clipboard.writeText(shareUrl).then(() => {
                showNotification('📋 Đã sao chép link chia sẻ!');
            });
        }

        function deleteFile(fileData, type) {
            if (confirm(`Bạn có chắc muốn xóa file ${fileData.name}?`)) {
                const index = fileDatabase[type].findIndex(f => f.id === fileData.id);
                if (index > -1) {
                    fileDatabase[type].splice(index, 1);
                    loadTabData(type);
                    showNotification(`🗑️ Đã xóa ${fileData.name}`);
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
            document.getElementById('inAnalysis').textContent = fileDatabase.analysis.filter(a => a.status !== 'completed').length;
            
            // Load sample connections
            loadSampleConnections();
            
            // Load recent activities
            loadRecentActivities();
        }

        function getTodaySamplesCount() {
            const today = new Date().toDateString();
            return fileDatabase.sampling.filter(s => 
                new Date(s.uploadDate).toDateString() === today
            ).length;
        }

        function loadSampleConnections() {
            const container = document.getElementById('sampleConnections');
            const samplesWithConnections = fileDatabase.sampling.filter(s => s.connections.length > 0);
            
            if (samplesWithConnections.length === 0) {
                container.innerHTML = '<p class="text-gray-500 col-span-3">Chưa có mẫu nào có mối liên hệ</p>';
                return;
            }
            
            container.innerHTML = samplesWithConnections.map(sample => `
                <div class="sample-connection p-4 rounded-lg">
                    <div class="flex items-center justify-between mb-2">
                        <h5 class="font-semibold text-gray-800">${sample.reference}</h5>
                        <span class="text-xs bg-blue-100 text-blue-800 px-2 py-1 rounded">${sample.customer}</span>
                    </div>
                    <p class="text-sm text-gray-600 mb-2">Liên hệ với: ${sample.connections.join(', ')}</p>
                    <div class="flex items-center text-xs text-gray-500">
                        <i class="fas fa-calendar mr-1"></i>
                        ${new Date(sample.uploadDate).toLocaleDateString('vi-VN')}
                    </div>
                </div>
            `).join('');
        }

        function loadRecentActivities() {
            const container = document.getElementById('recentActivities');
            const allFiles = [
                ...fileDatabase.contracts.map(f => ({...f, type: 'contract', icon: 'fa-file-contract', color: 'blue'})),
                ...fileDatabase.sampling.map(f => ({...f, type: 'sampling', icon: 'fa-vial', color: 'green'})),
                ...fileDatabase.analysis.map(f => ({...f, type: 'analysis', icon: 'fa-microscope', color: 'orange'})),
                ...fileDatabase.results.map(f => ({...f, type: 'results', icon: 'fa-chart-line', color: 'purple'}))
            ].sort((a, b) => new Date(b.uploadDate) - new Date(a.uploadDate)).slice(0, 5);
            
            if (allFiles.length === 0) {
                container.innerHTML = '<p class="text-gray-500">Chưa có hoạt động nào</p>';
                return;
            }
            
            container.innerHTML = allFiles.map(file => `
                <div class="flex items-center p-4 bg-${file.color}-50 rounded-lg">
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
                tbody.innerHTML = '<tr><td colspan="6" class="text-center py-8 text-gray-500">Chưa có hợp đồng nào</td></tr>';
                return;
            }
            
            tbody.innerHTML = fileDatabase.contracts.map(contract => `
                <tr class="hover:bg-gray-50">
                    <td class="px-4 py-3">${contract.reference}</td>
                    <td class="px-4 py-3">${contract.customer}</td>
                    <td class="px-4 py-3">${new Date(contract.uploadDate).toLocaleDateString('vi-VN')}</td>
                    <td class="px-4 py-3">
                        <div class="flex items-center space-x-2">
                            <i class="fas ${getFileIcon(contract.type)} text-${contract.type === 'word' ? 'blue' : contract.type === 'pdf' ? 'red' : 'green'}-600"></i>
                            <span>${contract.name}</span>
                            <span class="text-xs bg-gray-100 px-2 py-1 rounded">${formatFileSize(contract.size)}</span>
                        </div>
                    </td>
                    <td class="px-4 py-3">
                        <span class="bg-green-100 text-green-800 px-2 py-1 rounded-full text-sm">Đang thực hiện</span>
                    </td>
                    <td class="px-4 py-3">
                        <button onclick="previewFile(${JSON.stringify(contract).replace(/"/g, '&quot;')})" class="text-blue-600 hover:text-blue-800 mr-2" title="Xem trước">
                            <i class="fas fa-eye"></i>
                        </button>
                        <button onclick="downloadFile(${JSON.stringify(contract).replace(/"/g, '&quot;')})" class="text-green-600 hover:text-green-800 mr-2" title="Tải xuống">
                            <i class="fas fa-download"></i>
                        </button>
                        <button onclick="shareFile(${JSON.stringify(contract).replace(/"/g, '&quot;')})" class="text-purple-600 hover:text-purple-800 mr-2" title="Chia sẻ">
                            <i class="fas fa-share-alt"></i>
                        </button>
                        <button onclick="deleteFile(${JSON.stringify(contract).replace(/"/g, '&quot;')}, 'contracts')" class="text-red-600 hover:text-red-800" title="Xóa">
                            <i class="fas fa-trash"></i>
                        </button>
                    </td>
                </tr>
            `).join('');
        }

        function loadSampling() {
            const tbody = document.getElementById('samplingTable');
            
            if (fileDatabase.sampling.length === 0) {
                tbody.innerHTML = '<tr><td colspan="8" class="text-center py-8 text-gray-500">Chưa có biên bản lấy mẫu nào</td></tr>';
                return;
            }
            
            tbody.innerHTML = fileDatabase.sampling.map(sample => `
                <tr class="hover:bg-gray-50">
                    <td class="px-4 py-3">${sample.reference}</td>
                    <td class="px-4 py-3">${sample.customer}</td>
                    <td class="px-4 py-3">${new Date(sample.uploadDate).toLocaleDateString('vi-VN')}</td>
                    <td class="px-4 py-3">Nước thải</td>
                    <td class="px-4 py-3">5 mẫu</td>
                    <td class="px-4 py-3">
                        <div class="flex items-center space-x-2">
                            <i class="fas ${getFileIcon(sample.type)} text-${sample.type === 'word' ? 'blue' : sample.type === 'pdf' ? 'red' : 'green'}-600"></i>
                            <span>${sample.name}</span>
                        </div>
                    </td>
                    <td class="px-4 py-3">
                        ${sample.connections.length > 0 ? 
                            `<span class="text-xs bg-purple-100 text-purple-800 px-2 py-1 rounded">${sample.connections.join(', ')}</span>` : 
                            '<span class="text-gray-400">Không có</span>'
                        }
                    </td>
                    <td class="px-4 py-3">
                        <button onclick="previewFile(${JSON.stringify(sample).replace(/"/g, '&quot;')})" class="text-blue-600 hover:text-blue-800 mr-2" title="Xem trước">
                            <i class="fas fa-eye"></i>
                        </button>
                        <button onclick="downloadFile(${JSON.stringify(sample).replace(/"/g, '&quot;')})" class="text-green-600 hover:text-green-800 mr-2" title="Tải xuống">
                            <i class="fas fa-download"></i>
                        </button>
                        <button onclick="shareFile(${JSON.stringify(sample).replace(/"/g, '&quot;')})" class="text-purple-600 hover:text-purple-800 mr-2" title="Chia sẻ">
                            <i class="fas fa-share-alt"></i>
                        </button>
                        <button onclick="deleteFile(${JSON.stringify(sample).replace(/"/g, '&quot;')}, 'sampling')" class="text-red-600 hover:text-red-800" title="Xóa">
                            <i class="fas fa-trash"></i>
                        </button>
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
                        <button onclick="generateQRCode('${code.reference}', '${code.customer}')" class="text-purple-600 hover:text-purple-800" title="In QR Code">
                            <i class="fas fa-qrcode"></i>
                        </button>
                    </div>
                </div>
            `).join('');
        }

        function loadAnalysis() {
            const tbody = document.getElementById('analysisTable');
            
            if (fileDatabase.analysis.length === 0) {
                tbody.innerHTML = '<tr><td colspan="7" class="text-center py-8 text-gray-500">Chưa có biên bản phân tích nào</td></tr>';
                return;
            }
            
            tbody.innerHTML = fileDatabase.analysis.map(analysis => `
                <tr class="hover:bg-gray-50">
                    <td class="px-4 py-3">${analysis.reference}</td>
                    <td class="px-4 py-3">${analysis.reference.replace(/PA/, 'MA')}</td>
                    <td class="px-4 py-3">${new Date(analysis.uploadDate).toLocaleDateString('vi-VN')}</td>
                    <td class="px-4 py-3">Nguyễn Văn A</td>
                    <td class="px-4 py-3">
                        <div class="flex items-center space-x-2">
                            <i class="fas ${getFileIcon(analysis.type)} text-${analysis.type === 'word' ? 'blue' : analysis.type === 'pdf' ? 'red' : 'green'}-600"></i>
                            <span>${analysis.name}</span>
                        </div>
                    </td>
                    <td class="px-4 py-3">
                        <span class="bg-orange-100 text-orange-800 px-2 py-1 rounded-full text-sm">Đang phân tích</span>
                    </td>
                    <td class="px-4 py-3">
                        <button onclick="previewFile(${JSON.stringify(analysis).replace(/"/g, '&quot;')})" class="text-blue-600 hover:text-blue-800 mr-2" title="Xem trước">
                            <i class="fas fa-eye"></i>
                        </button>
                        <button onclick="downloadFile(${JSON.stringify(analysis).replace(/"/g, '&quot;')})" class="text-green-600 hover:text-green-800 mr-2" title="Tải xuống">
                            <i class="fas fa-download"></i>
                        </button>
                        <button onclick="shareFile(${JSON.stringify(analysis).replace(/"/g, '&quot;')})" class="text-purple-600 hover:text-purple-800 mr-2" title="Chia sẻ">
                            <i class="fas fa-share-alt"></i>
                        </button>
                        <button onclick="deleteFile(${JSON.stringify(analysis).replace(/"/g, '&quot;')}, 'analysis')" class="text-red-600 hover:text-red-800" title="Xóa">
                            <i class="fas fa-trash"></i>
                        </button>
                    </td>
                </tr>
            `).join('');
        }

        function loadResults() {
            const tbody = document.getElementById('resultsTable');
            
            if (fileDatabase.results.length === 0) {
                tbody.innerHTML = '<tr><td colspan="7" class="text-center py-8 text-gray-500">Chưa có kết quả thử nghiệm nào</td></tr>';
                return;
            }
            
            tbody.innerHTML = fileDatabase.results.map(result => `
                <tr class="hover:bg-gray-50">
                    <td class="px-4 py-3">${result.reference}</td>
                    <td class="px-4 py-3">${result.reference.replace(/KQ/, 'MA')}</td>
                    <td class="px-4 py-3">${result.customer}</td>
                    <td class="px-4 py-3">${new Date(result.uploadDate).toLocaleDateString('vi-VN')}</td>
                    <td class="px-4 py-3">
                        <div class="flex items-center space-x-2">
                            <i class="fas ${getFileIcon(result.type)} text-${result.type === 'word' ? 'blue' : result.type === 'pdf' ? 'red' : 'green'}-600"></i>
                            <span>${result.name}</span>
                        </div>
                    </td>
                    <td class="px-4 py-3">
                        <span class="bg-green-100 text-green-800 px-2 py-1 rounded-full text-sm">Đạt chuẩn</span>
                    </td>
                    <td class="px-4 py-3">
                        <button onclick="previewFile(${JSON.stringify(result).replace(/"/g, '&quot;')})" class="text-blue-600 hover:text-blue-800 mr-2" title="Xem trước">
                            <i class="fas fa-eye"></i>
                        </button>
                        <button onclick="downloadFile(${JSON.stringify(result).replace(/"/g, '&quot;')})" class="text-green-600 hover:text-green-800 mr-2" title="Tải xuống">
                            <i class="fas fa-download"></i>
                        </button>
                        <button onclick="shareFile(${JSON.stringify(result).replace(/"/g, '&quot;')})" class="text-purple-600 hover:text-purple-800 mr-2" title="Gửi khách hàng">
                            <i class="fas fa-paper-plane"></i>
                        </button>
                        <button onclick="deleteFile(${JSON.stringify(result).replace(/"/g, '&quot;')}, 'results')" class="text-red-600 hover:text-red-800" title="Xóa">
                            <i class="fas fa-trash"></i>
                        </button>
                    </td>
                </tr>
            `).join('');
        }

        function loadCustomers() {
            const customerList = document.getElementById('customerList');
            const customerFilter = document.getElementById('customerFilter');
            
            // Get unique customers
            const allFiles = [
                ...fileDatabase.contracts,
                ...fileDatabase.sampling,
                ...fileDatabase.analysis,
                ...fileDatabase.results
            ];
            
            const customers = [...new Set(allFiles.map(f => f.customer))];
            
            // Update filter dropdown
            customerFilter.innerHTML = '<option value="">Tất cả khách hàng</option>' + 
                customers.map(customer => `<option value="${customer}">${customer}</option>`).join('');
            
            // Display customer list
            if (customers.length === 0) {
                customerList.innerHTML = '<p class="text-gray-500">Chưa có khách hàng nào</p>';
                return;
            }
            
            customerList.innerHTML = customers.map(customer => {
                const customerFiles = allFiles.filter(f => f.customer === customer);
                const contractCount = customerFiles.filter(f => fileDatabase.contracts.includes(f)).length;
                const sampleCount = customerFiles.filter(f => fileDatabase.sampling.includes(f)).length;
                const resultCount = customerFiles.filter(f => fileDatabase.results.includes(f)).length;
                
                return `
                    <div class="border border-gray-200 rounded-lg p-4 hover:shadow-md transition-shadow cursor-pointer" onclick="showCustomerProgress('${customer}')">
                        <div class="flex justify-between items-start">
                            <div>
                                <h5 class="font-semibold text-gray-800">${customer}</h5>
                                <p class="text-sm text-gray-600">Tổng: ${customerFiles.length} file</p>
                                <div class="flex space-x-4 mt-2 text-xs text-gray-500">
                                    <span>HĐ: ${contractCount}</span>
                                    <span>Mẫu: ${sampleCount}</span>
                                    <span>KQ: ${resultCount}</span>
                                </div>
                            </div>
                            <div class="text-right">
                                <span class="bg-blue-100 text-blue-800 px-2 py-1 rounded text-sm">${customerFiles.length} file</span>
                            </div>
                        </div>
                    </div>
                `;
            }).join('');
        }

        function showCustomerProgress(customer) {
            const progressDiv = document.getElementById('customerProgress');
            
            const allFiles = [
                ...fileDatabase.contracts,
                ...fileDatabase.sampling,
                ...fileDatabase.analysis,
                ...fileDatabase.results
            ];
            
            const customerFiles = allFiles.filter(f => f.customer === customer);
            
            if (customerFiles.length === 0) {
                progressDiv.innerHTML = '<p class="text-gray-500">Không có dữ liệu</p>';
                return;
            }
            
            const progress = {
                contracts: customerFiles.filter(f => fileDatabase.contracts.includes(f)).length,
                sampling: customerFiles.filter(f => fileDatabase.sampling.includes(f)).length,
                analysis: customerFiles.filter(f => fileDatabase.analysis.includes(f)).length,
                results: customerFiles.filter(f => fileDatabase.results.includes(f)).length
            };
            
            const total = Object.values(progress).reduce((a, b) => a + b, 0);
            
            progressDiv.innerHTML = `
                <div class="text-left">
                    <h5 class="font-semibold text-gray-800 mb-3">${customer}</h5>
                    <div class="space-y-3">
                        <div class="flex justify-between items-center">
                            <span class="text-sm">Hợp đồng</span>
                            <span class="font-semibold">${progress.contracts}</span>
                        </div>
                        <div class="w-full bg-gray-200 rounded-full h-2">
                            <div class="bg-blue-600 h-2 rounded-full" style="width: ${(progress.contracts/total)*100}%"></div>
                        </div>
                        
                        <div class="flex justify-between items-center">
                            <span class="text-sm">Lấy mẫu</span>
                            <span class="font-semibold">${progress.sampling}</span>
                        </div>
                        <div class="w-full bg-gray-200 rounded-full h-2">
                            <div class="bg-green-600 h-2 rounded-full" style="width: ${(progress.sampling/total)*100}%"></div>
                        </div>
                        
                        <div class="flex justify-between items-center">
                            <span class="text-sm">Phân tích</span>
                            <span class="font-semibold">${progress.analysis}</span>
                        </div>
                        <div class="w-full bg-gray-200 rounded-full h-2">
                            <div class="<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9663b6c6b326ddc8',t:'MTc1MzY5ODc2MS4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script>
