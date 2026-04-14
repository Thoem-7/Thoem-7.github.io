# Thoem-7.github.io
Sales Dasboard Executive Summary
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Sales Dashboard Executive Summary : OB2 & HOR</title>
    
    <!-- Fix: Define process for libraries that expect node env -->
    <script>
        window.process = { env: { NODE_ENV: 'production' } };
    </script>

    <!-- 1. React & ReactDOM (Ver 18.2.0) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js" crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js" crossorigin="anonymous"></script>
    
    <!-- 2. Prop Types -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/prop-types/15.8.1/prop-types.min.js" crossorigin="anonymous"></script>

    <!-- 3. Recharts (Ver 2.12.7) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/recharts/2.12.7/Recharts.min.js" crossorigin="anonymous"></script>
    
    <!-- 4. Babel -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.5/babel.min.js" crossorigin="anonymous"></script>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>

    <!-- Font -->
    <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;500;700;800&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Sarabun', sans-serif; -webkit-tap-highlight-color: transparent; background-color: #f8fafc; }
        .custom-scrollbar::-webkit-scrollbar { height: 6px; width: 6px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: #f1f5f9; border-radius: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
        select, button, input { touch-action: manipulation; }
        
        /* Glassmorphism utility */
        .glass-card {
            background: rgba(255, 255, 255, 0.95);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.2);
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05), 0 2px 4px -1px rgba(0, 0, 0, 0.03);
        }
        
        /* Loading Spinner */
        .loader {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #3b82f6;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body class="text-slate-800">

    <div id="root"></div>

    <script type="text/babel">
        const { useState, useMemo, useEffect } = React;
        const Recharts = window.Recharts;
        if (!Recharts) {
            document.getElementById('root').innerHTML = `
                <div class="flex items-center justify-center min-h-screen">
                    <div class="p-6 bg-red-50 text-red-700 rounded-lg border border-red-200 shadow-md">
                        <h3 class="font-bold text-lg mb-2">Error Loading Dashboard</h3>
                        <p>ไม่สามารถโหลด Recharts Library ได้ กรุณาตรวจสอบการเชื่อมต่ออินเทอร์เน็ต</p>
                    </div>
                </div>
            `;
            throw new Error("Recharts not loaded");
        }

        const { LineChart, Line, BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer, ComposedChart } = Recharts;

        // --- Inline Icons (SVG) ---
        const Icon = ({ path, className, size = 24 }) => (
            <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}>
                {path}
            </svg>
        );

        const Icons = {
            Upload: (props) => <Icon {...props} path={<><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></>} />,
            Filter: (props) => <Icon {...props} path={<><polygon points="22 3 2 3 10 12.46 10 19 14 21 14 12.46 22 3"/></>} />,
            Target: (props) => <Icon {...props} path={<><circle cx="12" cy="12" r="10"/><circle cx="12" cy="12" r="6"/><circle cx="12" cy="12" r="2"/></>} />,
            Dollar: (props) => <Icon {...props} path={<><line x1="12" y1="1" x2="12" y2="23"/><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/></>} />,
            Award: (props) => <Icon {...props} path={<><circle cx="12" cy="8" r="7"/><polyline points="8.21 13.89 7 23 12 20 17 23 15.79 13.88"/></>} />,
            Store: (props) => <Icon {...props} path={<><path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/><polyline points="9 22 9 12 15 12 15 22"/></>} />,
            Package: (props) => <Icon {...props} path={<><line x1="16.5" y1="9.4" x2="7.5" y2="4.21"/><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></>} />,
            TrendingUp: (props) => <Icon {...props} path={<><polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/><polyline points="17 6 23 6 23 12"/></>} />,
            Table: (props) => <Icon {...props} path={<><rect width="18" height="18" x="3" y="3" rx="2" ry="2"/><line x1="3" x2="21" y1="9" y2="9"/><line x1="3" x2="21" y1="15" y2="15"/><line x1="12" x2="12" y1="3" y2="21"/></>} />,
            ChevronLeft: (props) => <Icon {...props} path={<><polyline points="15 18 9 12 15 6"/></>} />,
            ChevronRight: (props) => <Icon {...props} path={<><polyline points="9 18 15 12 9 6"/></>} />,
            Refresh: (props) => <Icon {...props} path={<><path d="M21.5 2v6h-6M21.34 15.57a10 10 0 1 1-.59-8.38l5.67-5.67"/></>} />
        };

        // --- Utility Functions ---
        const splitCSV = (str) => {
            const row = []; let cur = ''; let inQuote = false;
            for(let i=0; i<str.length; i++){
                let char = str[i];
                if(char === '"' && str[i+1] === '"') { cur += '"'; i++; } 
                else if(char === '"') inQuote = !inQuote;
                else if(char === ',' && !inQuote) { row.push(cur); cur = ''; }
                else cur += char;
            }
            row.push(cur);
            return row;
        };

        const cleanNumber = (val) => {
            if (!val) return 0;
            let str = String(val).replace(/["$,]/g, '').trim();
            if (str === '-' || str === '') return 0;
            return parseFloat(str) || 0;
        };

        const calculateAch = (sales, target) => {
            if (target > 0) return ((sales / target) * 100).toFixed(1);
            if (sales > 0 && target === 0) return "100.0";
            return "0.0";
        };

        const formatTHB = (val) => new Intl.NumberFormat('th-TH', { style: 'currency', currency: 'THB', minimumFractionDigits: 0, maximumFractionDigits: 0 }).format(val);
        const formatNumber = (val) => new Intl.NumberFormat('th-TH', { maximumFractionDigits: 0 }).format(val);
        
        const MONTH_NAMES_ENG = ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"];
        const MONTH_NAMES_TH = ["มกราคม", "กุมภาพันธ์", "มีนาคม", "เมษายน", "พฤษภาคม", "มิถุนายน", "กรกฎาคม", "สิงหาคม", "กันยายน", "ตุลาคม", "พฤศจิกายน", "ธันวาคม"];

        // Tooltip Customization
        const CustomTooltip = ({ active, payload, label }) => {
            if (active && payload && payload.length) {
                const ach = payload[0].payload.Ach;
                return (
                    <div className="bg-white p-3 border border-slate-200 shadow-xl rounded-xl z-50 relative">
                        <p className="font-bold text-slate-800 mb-2 border-b border-slate-100 pb-2">{label}</p>
                        {payload.map((entry, index) => (
                            <p key={index} style={{ color: entry.color }} className="text-sm font-semibold flex justify-between gap-4">
                                <span>{entry.name}:</span> 
                                <span>{formatNumber(entry.value)}</span>
                            </p>
                        ))}
                        {ach !== undefined && (
                            <div className={`mt-3 pt-2 border-t border-slate-100 text-sm font-extrabold flex justify-between ${ach >= 100 ? 'text-emerald-600' : 'text-rose-500'}`}>
                                <span>% Ach:</span>
                                <span>{ach}%</span>
                            </div>
                        )}
                    </div>
                );
            }
            return null;
        };

        // --- Main Component ---
        const SalesDashboard = () => {
            // States
            const [data, setData] = useState([]);
            const [outletData, setOutletData] = useState({});
            const [loadedFiles, setLoadedFiles] = useState([]);
            const [isLoading, setIsLoading] = useState(true); // Loading State for API
            const [apiError, setApiError] = useState(false);
            
            // Filters
            const [selectedYear, setSelectedYear] = useState('All');
            const [selectedQuarter, setSelectedQuarter] = useState('All'); 
            const [selectedMonth, setSelectedMonth] = useState('All');
            const [selectedTeam, setSelectedTeam] = useState('All');
            const [selectedArea, setSelectedArea] = useState('All'); 
            const [selectedSupCode, setSelectedSupCode] = useState('All');
            
            const [selectedGroupSales, setSelectedGroupSales] = useState('All');
            const [selectedPrdType, setSelectedPrdType] = useState('All');
            const [selectedProductName, setSelectedProductName] = useState('All');

            // Pagination
            const [currentPage, setCurrentPage] = useState(1);
            const itemsPerPage = 15;

            // --- Core Parsing Function ---
            const processFilesData = (fileResults) => {
                let consolidated = {};
                let newOutlet = {}; // Reset state for clean load
                let fileNames = [];

                fileResults.forEach(({ name, text }) => {
                    fileNames.push(name);

                    // 1. Detect Executive Summary (Outlet Data)
                    if (text.includes('Outlet Target') || text.includes('Outlet (Active)')) {
                        const lines = text.split('\n').map(l => l.trim());
                        let mode = null;
                        lines.forEach(line => {
                            const cols = splitCSV(line).map(c => c.trim().replace(/["']/g, ''));
                            if (cols.includes('Outlet Target')) {
                                if (cols.includes('SALES_CODE')) mode = 'SALES_CODE';
                                return;
                            }
                            if (mode === 'SALES_CODE' && cols.length >= 8 && cols[2]) {
                                let salesCode = cols[2];
                                if (salesCode.startsWith('OB') || salesCode.startsWith('H')) {
                                    newOutlet[salesCode] = {
                                        target: cleanNumber(cols[6]),
                                        active: cleanNumber(cols[7])
                                    };
                                }
                            }
                        });
                    } 
                    // 2. Detect Detailed Sales / Target Data
                    else if (text.includes('SM_CODE_NEW')) {
                        const lines = text.split('\n').map(l => l.trim()).filter(l => l);
                        const headers = splitCSV(lines[0]).map(c => c.trim().replace(/["']/g, ''));
                        
                        const idxTeam = headers.indexOf('SM_CODE_NEW');
                        const idxSup = headers.indexOf('SUP');
                        const idxSalesCode = headers.indexOf('SALES_CODE');
                        const idxYear = headers.indexOf('Year');
                        const idxQuarter = headers.indexOf('Quarter'); 
                        const idxMonth = headers.indexOf('Month');
                        const idxGroup = headers.indexOf('Group_Sales');
                        const idxPrd = headers.indexOf('PRD Type');
                        const idxName = headers.indexOf('PRODUCT_NAME_NEW');
                        
                        // Check for Target or Sales column dynamically
                        const idxSales = headers.findIndex(h => h.toLowerCase() === 'sales');
                        const idxTarget = headers.findIndex(h => h.toLowerCase() === 'target');
                        const idxAmount = headers.findIndex(h => h.toLowerCase() === 'amount');
                        const idxType = headers.findIndex(h => h.toLowerCase() === 'type');

                        for(let i=1; i<lines.length; i++) {
                            const row = splitCSV(lines[i]);
                            if (row.length === headers.length) {
                                let salesVal = 0, targetVal = 0;

                                // V03 explicit structure
                                if (idxSales >= 0) salesVal = cleanNumber(row[idxSales]);
                                if (idxTarget >= 0) targetVal = cleanNumber(row[idxTarget]);

                                // V02 old structure fallback
                                if (idxType >= 0 && idxAmount >= 0) {
                                    let type = row[idxType].toUpperCase();
                                    if (type === 'SALES') salesVal = cleanNumber(row[idxAmount]);
                                    if (type === 'TARGET') targetVal = cleanNumber(row[idxAmount]);
                                }

                                if (salesVal === 0 && targetVal === 0) continue;

                                let team = row[idxTeam];
                                let sup = row[idxSup];
                                let salesCode = row[idxSalesCode];
                                let year = parseInt(row[idxYear]) || 2026;
                                let quarter = row[idxQuarter] ? row[idxQuarter].trim() : '';
                                let month = parseInt(row[idxMonth]) || 1;
                                let group = row[idxGroup];
                                let prdType = row[idxPrd];
                                let productName = row[idxName];

                                let key = `${team}|${sup}|${salesCode}|${year}|${quarter}|${month}|${group}|${prdType}|${productName}`;
                                
                                if (!consolidated[key]) {
                                    consolidated[key] = { 
                                        team, sup, salesCode, year, quarter, month, group, prdType, productName, 
                                        target: 0, sales: 0 
                                    };
                                }
                                consolidated[key].target += targetVal;
                                consolidated[key].sales += salesVal;
                            }
                        }
                    }
                });
                
                setData(Object.values(consolidated));
                setOutletData(newOutlet);
                setLoadedFiles(fileNames);
                setCurrentPage(1);
            };

            // --- AUTO-FETCH API (Load Data without Upload) ---
            const fetchAPIData = async () => {
                setIsLoading(true);
                setApiError(false);
                try {
                    const apiUrls = [
                        'Executive Summary_OB2&HOR_26Y04_wk_02-2026.csv',
                        'Sales_HOR_V03.csv',
                        'Sales_OB2_V03.csv',
                        'Target_HOR_V03.csv',
                        'Target_OB2_V03.csv'
                    ];

                    const fetchPromises = apiUrls.map(async (url) => {
                        const res = await fetch(url, { cache: "no-store" });
                        if (!res.ok) throw new Error(`HTTP error! status: ${res.status}`);
                        const text = await res.text();
                        return { name: url.split('/').pop(), text: text };
                    });

                    const fileResults = await Promise.all(fetchPromises);
                    processFilesData(fileResults);
                } catch (err) {
                    console.log("Auto-fetch API ไม่สำเร็จ (เปิด Offline หรือไม่มีไฟล์บน Server)", err);
                    setApiError(true);
                } finally {
                    setIsLoading(false);
                }
            };

            useEffect(() => {
                fetchAPIData();
            }, []);

            // --- Manual File Upload Fallback ---
            const handleFileUpload = async (event) => {
                const files = Array.from(event.target.files);
                if (files.length === 0) return;

                const readAsText = (file) => new Promise((resolve) => {
                    const reader = new FileReader();
                    reader.onload = (e) => resolve({ name: file.name, text: e.target.result });
                    reader.readAsText(file);
                });

                const fileResults = await Promise.all(files.map(readAsText));
                processFilesData(fileResults);
            };

            // Filter Options
            const filterOptions = useMemo(() => {
                const years = [...new Set(data.map(d => d.year))].filter(Boolean).sort();
                const quarters = [...new Set(data.map(d => d.quarter))].filter(Boolean).sort();
                const teams = [...new Set(data.map(d => d.team))].filter(Boolean).sort();
                const supCodes = [...new Set(data.map(d => d.sup))].filter(Boolean).sort();
                
                const groupSales = [...new Set(data.map(d => d.group))].filter(Boolean).sort();
                const prdTypes = [...new Set(data.map(d => d.prdType))].filter(Boolean).sort();
                const productNames = [...new Set(data.map(d => d.productName))].filter(Boolean).sort();

                const areas = [...new Set(data.filter(d => selectedTeam === 'All' || d.team === selectedTeam).map(d => d.salesCode))].filter(Boolean).sort();
                
                return { years, quarters, teams, supCodes, areas, groupSales, prdTypes, productNames };
            }, [data, selectedTeam]);

            // Apply Filters
            const filteredData = useMemo(() => {
                return data.filter(item => {
                    const yearMatch = selectedYear === 'All' || item.year === parseInt(selectedYear);
                    const quarterMatch = selectedQuarter === 'All' || item.quarter === selectedQuarter;
                    const monthMatch = selectedMonth === 'All' || item.month === parseInt(selectedMonth);
                    const teamMatch = selectedTeam === 'All' || item.team === selectedTeam;
                    const areaMatch = selectedArea === 'All' || item.salesCode === selectedArea;
                    const supMatch = selectedSupCode === 'All' || item.sup === selectedSupCode;
                    
                    const groupMatch = selectedGroupSales === 'All' || item.group === selectedGroupSales;
                    const prdTypeMatch = selectedPrdType === 'All' || item.prdType === selectedPrdType;
                    const productNameMatch = selectedProductName === 'All' || item.productName === selectedProductName;

                    return yearMatch && quarterMatch && monthMatch && teamMatch && areaMatch && supMatch && groupMatch && prdTypeMatch && productNameMatch;
                });
            }, [data, selectedYear, selectedQuarter, selectedMonth, selectedTeam, selectedArea, selectedSupCode, selectedGroupSales, selectedPrdType, selectedProductName]);

            // KPIs Calculation
            const kpis = useMemo(() => {
                let totalSales = 0, totalTarget = 0;
                filteredData.forEach(d => {
                    totalSales += d.sales;
                    totalTarget += d.target;
                });

                let uniqueSalesCodes = [...new Set(filteredData.map(d => d.salesCode))];
                let outTarget = 0, outActive = 0;
                uniqueSalesCodes.forEach(code => {
                    if (outletData[code]) {
                        outTarget += outletData[code].target;
                        outActive += outletData[code].active;
                    }
                });

                return { 
                    totalSales, 
                    totalTarget, 
                    salesAch: calculateAch(totalSales, totalTarget),
                    outTarget,
                    outActive,
                    outAch: calculateAch(outActive, outTarget)
                };
            }, [filteredData, outletData]);

            // Topics Analysis
            const topicsAnalysis = useMemo(() => {
                const res = {
                    pushOther: { sales: 0, target: 0 },
                    existing: { sales: 0, target: 0 },
                    newGroup: { sales: 0, target: 0 }
                };

                filteredData.forEach(d => {
                    const isPushOther = (d.prdType === 'Product Push' || d.prdType === 'Other Product');
                    const isExisting = (d.group === 'Existing Product' || d.group === 'Existing Products');
                    const isNew = (d.group === 'New Product' || d.prdType === 'New Product');

                    if (isPushOther) { res.pushOther.sales += d.sales; res.pushOther.target += d.target; }
                    if (isExisting) { res.existing.sales += d.sales; res.existing.target += d.target; }
                    if (isNew) { res.newGroup.sales += d.sales; res.newGroup.target += d.target; }
                });

                return res;
            }, [filteredData]);

            // Executive Summary Table Data
            const execSummaryData = useMemo(() => {
                const grouped = {};
                filteredData.forEach(d => {
                    let key = `${d.team}|${d.sup}|${d.salesCode}`;
                    if (!grouped[key]) {
                        grouped[key] = { team: d.team, sup: d.sup, area: d.salesCode, sales: 0, target: 0 };
                    }
                    grouped[key].sales += d.sales;
                    grouped[key].target += d.target;
                });

                return Object.values(grouped).map(g => {
                    const out = outletData[g.area] || { target: 0, active: 0 };
                    return {
                        ...g,
                        salesAch: calculateAch(g.sales, g.target),
                        outTarget: out.target,
                        outActive: out.active,
                        outAch: calculateAch(out.active, out.target)
                    };
                }).sort((a, b) => a.team.localeCompare(b.team) || a.area.localeCompare(b.area));
            }, [filteredData, outletData]);

            // Area Comparison Chart Data
            const areaChartData = useMemo(() => {
                const grouped = {};
                filteredData.forEach(d => {
                    let key = selectedTeam === 'All' ? d.team : d.salesCode;
                    if (!grouped[key]) grouped[key] = { name: key, Sales: 0, Target: 0 };
                    grouped[key].Sales += d.sales;
                    grouped[key].Target += d.target;
                });
                return Object.values(grouped).map(d => ({
                    ...d, Ach: calculateAch(d.Sales, d.Target)
                })).sort((a, b) => b.Sales - a.Sales);
            }, [filteredData, selectedTeam]);

            // Monthly Trend Chart Data
            const trendData = useMemo(() => {
                const grouped = {};
                MONTH_NAMES_ENG.forEach((m, i) => { grouped[i+1] = { name: m, Sales: 0, Target: 0 }; });
                
                data.forEach(d => {
                    const teamMatch = selectedTeam === 'All' || d.team === selectedTeam;
                    const areaMatch = selectedArea === 'All' || d.salesCode === selectedArea;
                    const yearMatch = selectedYear === 'All' || d.year === parseInt(selectedYear);
                    const quarterMatch = selectedQuarter === 'All' || d.quarter === selectedQuarter;
                    const supMatch = selectedSupCode === 'All' || d.sup === selectedSupCode;
                    const groupMatch = selectedGroupSales === 'All' || d.group === selectedGroupSales;
                    const prdTypeMatch = selectedPrdType === 'All' || d.prdType === selectedPrdType;
                    const productNameMatch = selectedProductName === 'All' || d.productName === selectedProductName;
                    
                    if(grouped[d.month] && teamMatch && areaMatch && yearMatch && quarterMatch && supMatch && groupMatch && prdTypeMatch && productNameMatch) {
                        grouped[d.month].Sales += d.sales;
                        grouped[d.month].Target += d.target;
                    }
                });
                return Object.values(grouped).filter(d => d.Sales > 0 || d.Target > 0); 
            }, [data, selectedYear, selectedQuarter, selectedTeam, selectedArea, selectedSupCode, selectedGroupSales, selectedPrdType, selectedProductName]);

            // Pagination
            const paginatedData = useMemo(() => {
                const startIndex = (currentPage - 1) * itemsPerPage;
                return filteredData.slice(startIndex, startIndex + itemsPerPage);
            }, [filteredData, currentPage]);
            const totalPages = Math.ceil(filteredData.length / itemsPerPage) || 1;

            // Reset Page on Filter Change
            React.useEffect(() => { setCurrentPage(1); }, [selectedTeam, selectedArea, selectedYear, selectedQuarter, selectedMonth, selectedSupCode, selectedGroupSales, selectedPrdType, selectedProductName]);

            // --- Render Loading Screen ---
            if (isLoading) {
                return (
                    <div className="flex flex-col items-center justify-center min-h-screen bg-slate-50">
                        <div className="loader mb-4"></div>
                        <h2 className="text-xl font-bold text-slate-800">กำลังดึงข้อมูลจากระบบ (API)...</h2>
                        <p className="text-slate-500 mt-2 text-sm">Please wait while data is loading</p>
                    </div>
                );
            }

            // --- Render Empty State (Fallback for Manual Upload) ---
            if (data.length === 0) {
                return (
                    <div className="flex flex-col items-center justify-center min-h-screen bg-gradient-to-br from-slate-50 to-blue-50 p-6 font-sans">
                        <div className="bg-white p-10 rounded-[2rem] shadow-2xl text-center max-w-lg w-full border border-slate-100">
                            <div className="bg-gradient-to-tr from-blue-600 to-indigo-500 w-20 h-20 rounded-[1.5rem] flex items-center justify-center mx-auto mb-6 shadow-lg shadow-blue-200 transform rotate-3">
                                <Icons.Upload className="text-white" size={36} />
                            </div>
                            <h1 className="text-3xl font-extrabold text-slate-800 mb-3 tracking-tight">Executive Dashboard</h1>
                            <p className="text-slate-500 mb-2 font-medium text-lg">OB2 & HOR Summary</p>
                            
                            {apiError && (
                                <div className="bg-amber-50 text-amber-700 p-3 rounded-xl mb-4 text-sm font-semibold border border-amber-200 text-left">
                                    <p>⚠️ ไม่พบไฟล์บน Server เพื่อโหลดอัตโนมัติ</p>
                                    <p className="text-xs font-normal mt-1">สามารถอัปโหลดไฟล์ด้วยตนเองด้านล่าง เพื่อดูข้อมูลได้เลยครับ</p>
                                </div>
                            )}

                            <p className="text-slate-400 mb-8 text-sm px-4">กรุณาอัปโหลดไฟล์ Sales (V03), Target (V03) และ Executive Summary <br/><b>*สามารถคลุมดำเลือกหลายไฟล์พร้อมกันได้*</b></p>
                            
                            <label className="cursor-pointer bg-slate-900 hover:bg-blue-700 text-white font-bold py-4 px-8 rounded-2xl transition-all duration-300 flex items-center justify-center gap-3 shadow-xl hover:shadow-blue-500/30 transform hover:-translate-y-1">
                                <Icons.Upload size={22} />
                                <span className="text-lg">อัปโหลดไฟล์ CSV (Manual)</span>
                                <input type="file" accept=".csv" multiple onChange={handleFileUpload} className="hidden" />
                            </label>
                        </div>
                    </div>
                );
            }

            // Sub-Component: Topic Card
            const TopicCard = ({ title, sales, target, icon, bgClass }) => {
                const ach = calculateAch(sales, target);
                const isSuccess = parseFloat(ach) >= 100;
                return (
                    <div className="glass-card p-5 rounded-2xl relative overflow-hidden group h-full">
                        <div className="flex justify-between items-start mb-4">
                            <div>
                                <h4 className="text-sm font-bold text-slate-500 uppercase tracking-wider mb-1">{title}</h4>
                                <div className="flex items-baseline gap-2">
                                    <span className="text-2xl font-extrabold text-slate-800">{formatTHB(sales)}</span>
                                </div>
                                <p className="text-xs font-semibold text-slate-400 mt-1">Target: {formatTHB(target)}</p>
                            </div>
                            <div className={`${bgClass} p-3 rounded-xl shadow-sm`}>{icon}</div>
                        </div>
                        
                        <div className="mt-4 pt-4 border-t border-slate-100 flex items-center justify-between">
                            <span className="text-sm font-bold text-slate-600">% Achievement</span>
                            <span className={`px-3 py-1 rounded-lg text-sm font-extrabold ${isSuccess ? 'bg-emerald-100 text-emerald-700' : 'bg-rose-100 text-rose-700'}`}>
                                {ach}%
                            </span>
                        </div>
                        <div className="absolute bottom-0 left-0 w-full h-1.5 bg-slate-100">
                            <div className={`h-full ${isSuccess ? 'bg-emerald-500' : 'bg-rose-500'}`} style={{ width: `${Math.min(ach, 100)}%` }}></div>
                        </div>
                    </div>
                );
            };

            return (
                <div className="min-h-screen bg-slate-50 font-sans pb-12 flex flex-col">
                    {/* Header */}
                    <nav className="bg-slate-900 shadow-xl sticky top-0 z-40">
                        <div className="max-w-[1600px] mx-auto px-4 sm:px-6 lg:px-8">
                            <div className="flex justify-between h-20 items-center">
                                <div className="flex items-center gap-4">
                                    <div className="bg-gradient-to-br from-blue-500 to-indigo-600 p-2.5 rounded-xl shadow-inner">
                                        <Icons.Award className="text-white w-7 h-7" />
                                    </div>
                                    <div>
                                        <h1 className="text-lg sm:text-2xl font-extrabold text-white tracking-wide">Sales Dashboard Executive Summary : OB2 & HOR</h1>
                                        <p className="text-xs sm:text-sm text-blue-200 font-medium">Data Synced • {loadedFiles.length} Sources</p>
                                    </div>
                                </div>
                                <div className="flex gap-3">
                                    <button onClick={fetchAPIData} className="hidden md:flex text-sm font-bold text-white bg-transparent hover:bg-white/10 px-3 py-2.5 rounded-xl transition-all items-center gap-2">
                                        <Icons.Refresh size={18} /> Refresh API
                                    </button>
                                    <label className="cursor-pointer text-sm font-bold text-slate-900 bg-white hover:bg-blue-50 px-4 py-2.5 rounded-xl transition-all flex items-center gap-2 shadow-sm">
                                        <Icons.Upload size={18} />
                                        <span className="hidden md:inline">อัปโหลดเพิ่ม</span>
                                        <input type="file" accept=".csv" multiple onChange={handleFileUpload} className="hidden" />
                                    </label>
                                </div>
                            </div>
                        </div>
                    </nav>

                    {/* Main Layout Area */}
                    <main className="max-w-[1600px] w-full mx-auto px-4 sm:px-6 lg:px-8 py-8 flex flex-col lg:flex-row gap-8 items-start flex-1">
                        
                        {/* Sidebar: Filters Section */}
                        <aside className="w-full lg:w-72 xl:w-80 flex-shrink-0">
                            <div className="glass-card p-6 rounded-3xl lg:sticky lg:top-28 max-h-none lg:max-h-[calc(100vh-8rem)] overflow-y-auto custom-scrollbar">
                                <div className="flex items-center gap-3 mb-6 pb-4 border-b border-slate-200">
                                    <div className="bg-blue-100 p-2.5 rounded-xl"><Icons.Filter size={20} className="text-blue-700"/></div>
                                    <span className="font-extrabold text-slate-800 uppercase tracking-wide text-lg">Filters</span>
                                </div>
                                
                                <div className="flex flex-col gap-4">
                                    {/* Time Filters */}
                                    <div>
                                        <label className="block text-[11px] font-bold text-slate-500 uppercase mb-1.5 ml-1">Year (ปี)</label>
                                        <select value={selectedYear} onChange={(e) => setSelectedYear(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-blue-500 block p-3 font-semibold shadow-sm transition">
                                            <option value="All">ทุกปี (All)</option>
                                            {filterOptions.years.map(y => <option key={y} value={y}>{y}</option>)}
                                        </select>
                                    </div>
                                    <div>
                                        <label className="block text-[11px] font-bold text-slate-500 uppercase mb-1.5 ml-1">Quarter (ไตรมาส)</label>
                                        <select value={selectedQuarter} onChange={(e) => setSelectedQuarter(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-blue-500 block p-3 font-semibold shadow-sm transition">
                                            <option value="All">ทุกไตรมาส (All)</option>
                                            {filterOptions.quarters.map(q => <option key={q} value={q}>{q}</option>)}
                                        </select>
                                    </div>
                                    <div>
                                        <label className="block text-[11px] font-bold text-slate-500 uppercase mb-1.5 ml-1">Month (เดือน)</label>
                                        <select value={selectedMonth} onChange={(e) => setSelectedMonth(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-blue-500 block p-3 font-semibold shadow-sm transition">
                                            <option value="All">ทุกเดือน (All)</option>
                                            {MONTH_NAMES_TH.map((m, index) => <option key={index} value={index+1}>{m}</option>)}
                                        </select>
                                    </div>

                                    {/* Hierarchy Filters */}
                                    <div>
                                        <label className="block text-[11px] font-bold text-slate-500 uppercase mb-1.5 ml-1 mt-2 border-t border-slate-100 pt-3">Team (ทีม)</label>
                                        <select value={selectedTeam} onChange={(e) => setSelectedTeam(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-blue-500 block p-3 font-semibold shadow-sm transition">
                                            <option value="All">ทุกทีม (All)</option>
                                            {filterOptions.teams.map(t => <option key={t} value={t}>{t}</option>)}
                                        </select>
                                    </div>
                                    <div>
                                        <label className="block text-[11px] font-bold text-slate-500 uppercase mb-1.5 ml-1">Area (เขตการขาย)</label>
                                        <select value={selectedArea} onChange={(e) => setSelectedArea(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-blue-500 block p-3 font-semibold shadow-sm transition">
                                            <option value="All">ทุกเขต (All)</option>
                                            {filterOptions.areas.map(a => <option key={a} value={a}>{a}</option>)}
                                        </select>
                                    </div>
                                    <div>
                                        <label className="block text-[11px] font-bold text-slate-500 uppercase mb-1.5 ml-1">SUP Code</label>
                                        <select value={selectedSupCode} onChange={(e) => setSelectedSupCode(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-blue-500 block p-3 font-semibold shadow-sm transition">
                                            <option value="All">ทุก SUP (All)</option>
                                            {filterOptions.supCodes.map(s => <option key={s} value={s}>{s}</option>)}
                                        </select>
                                    </div>

                                    {/* Product Filters */}
                                    <div>
                                        <label className="block text-[11px] font-bold text-slate-500 uppercase mb-1.5 ml-1 mt-2 border-t border-slate-100 pt-3">Group Sales</label>
                                        <select value={selectedGroupSales} onChange={(e) => setSelectedGroupSales(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-blue-500 block p-3 font-semibold shadow-sm transition">
                                            <option value="All">ทุก Group (All)</option>
                                            {filterOptions.groupSales.map(c => <option key={c} value={c}>{c}</option>)}
                                        </select>
                                    </div>
                                    <div>
                                        <label className="block text-[11px] font-bold text-slate-500 uppercase mb-1.5 ml-1">PRD Type</label>
                                        <select value={selectedPrdType} onChange={(e) => setSelectedPrdType(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-blue-500 block p-3 font-semibold shadow-sm transition">
                                            <option value="All">ทุก PRD Type (All)</option>
                                            {filterOptions.prdTypes.map(c => <option key={c} value={c}>{c}</option>)}
                                        </select>
                                    </div>
                                    <div>
                                        <label className="block text-[11px] font-bold text-slate-500 uppercase mb-1.5 ml-1">Product Name</label>
                                        <select value={selectedProductName} onChange={(e) => setSelectedProductName(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-blue-500 block p-3 font-semibold shadow-sm transition">
                                            <option value="All">ทุกสินค้า (All)</option>
                                            {filterOptions.productNames.map(c => <option key={c} value={c}>{c}</option>)}
                                        </select>
                                    </div>
                                </div>
                            </div>
                        </aside>

                        {/* Right Area: Content & Data */}
                        <div className="flex-1 space-y-8 min-w-0 w-full">
                            
                            {/* Master KPIs */}
                            <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                                {/* Sales KPI */}
                                <div className="glass-card p-6 rounded-3xl flex items-center justify-between relative overflow-hidden border-l-8 border-l-blue-500">
                                    <div className="z-10">
                                        <p className="text-slate-500 text-sm font-extrabold uppercase tracking-widest mb-1">Target vs Sales</p>
                                        <div className="flex items-baseline gap-3 mt-2 flex-wrap">
                                            <h3 className="text-4xl font-black text-slate-800">{formatTHB(kpis.totalSales)}</h3>
                                            <span className="text-lg font-bold text-slate-400">/ {formatTHB(kpis.totalTarget)}</span>
                                        </div>
                                        <div className="mt-4 inline-flex items-center gap-2 bg-slate-50 px-3 py-1.5 rounded-lg border border-slate-100">
                                            <span className="text-xs font-bold text-slate-500 uppercase">Achievement</span>
                                            <span className={`text-lg font-black ${kpis.salesAch >= 100 ? 'text-emerald-600' : 'text-rose-600'}`}>{kpis.salesAch}%</span>
                                        </div>
                                    </div>
                                    <Icons.Dollar size={80} className="text-blue-50 opacity-50 absolute -right-4 -bottom-4 transform -rotate-12" />
                                </div>

                                {/* Outlet KPI */}
                                <div className="glass-card p-6 rounded-3xl flex items-center justify-between relative overflow-hidden border-l-8 border-l-amber-500">
                                    <div className="z-10">
                                        <p className="text-slate-500 text-sm font-extrabold uppercase tracking-widest mb-1">Outlet Target vs Outlet (Active)</p>
                                        <div className="flex items-baseline gap-3 mt-2 flex-wrap">
                                            <h3 className="text-4xl font-black text-slate-800">{formatNumber(kpis.outActive)}</h3>
                                            <span className="text-lg font-bold text-slate-400">/ {formatNumber(kpis.outTarget)}</span>
                                        </div>
                                        <div className="mt-4 inline-flex items-center gap-2 bg-slate-50 px-3 py-1.5 rounded-lg border border-slate-100">
                                            <span className="text-xs font-bold text-slate-500 uppercase">Achievement</span>
                                            <span className={`text-lg font-black ${kpis.outAch >= 100 ? 'text-emerald-600' : 'text-amber-600'}`}>{kpis.outAch}%</span>
                                        </div>
                                    </div>
                                    <Icons.Store size={80} className="text-amber-50 opacity-50 absolute -right-4 -bottom-4 transform rotate-12" />
                                </div>
                            </div>

                            {/* Executive Summary Table */}
                            <div className="glass-card rounded-3xl overflow-hidden shadow-sm">
                                <div className="p-6 border-b border-slate-100 flex items-center gap-3 bg-slate-50/80">
                                    <div className="bg-indigo-100 p-2 rounded-lg"><Icons.Table size={20} className="text-indigo-600"/></div>
                                    <h3 className="text-xl font-extrabold text-slate-800">Executive Summary By Team / Area</h3>
                                </div>
                                <div className="overflow-x-auto custom-scrollbar bg-white">
                                    <table className="w-full text-sm text-left text-slate-600">
                                        <thead className="text-xs text-slate-500 uppercase bg-slate-50 whitespace-nowrap">
                                            <tr>
                                                <th className="px-6 py-4 font-extrabold tracking-wider border-r border-slate-100">Team</th>
                                                <th className="px-6 py-4 font-extrabold tracking-wider border-r border-slate-100">SUP Code</th>
                                                <th className="px-6 py-4 font-extrabold tracking-wider border-r border-slate-200">Area (Sales Code)</th>
                                                <th className="px-6 py-4 font-extrabold tracking-wider text-right text-blue-600">Target</th>
                                                <th className="px-6 py-4 font-extrabold tracking-wider text-right text-blue-600">Sales</th>
                                                <th className="px-6 py-4 font-extrabold tracking-wider text-center border-r border-slate-200 text-blue-600">% Ach</th>
                                                <th className="px-6 py-4 font-extrabold tracking-wider text-right text-amber-600">Outlet Target</th>
                                                <th className="px-6 py-4 font-extrabold tracking-wider text-right text-amber-600">Outlet (Active)</th>
                                                <th className="px-6 py-4 font-extrabold tracking-wider text-center text-amber-600">% Ach</th>
                                            </tr>
                                        </thead>
                                        <tbody className="divide-y divide-slate-100">
                                            {execSummaryData.length > 0 ? execSummaryData.map((item, idx) => {
                                                const salesSuccess = parseFloat(item.salesAch) >= 100;
                                                const outSuccess = parseFloat(item.outAch) >= 100;
                                                return (
                                                    <tr key={idx} className="hover:bg-slate-50 transition-colors whitespace-nowrap">
                                                        <td className="px-6 py-4 font-black text-slate-700 border-r border-slate-100">{item.team}</td>
                                                        <td className="px-6 py-4 font-medium text-slate-500 border-r border-slate-100">{item.sup}</td>
                                                        <td className="px-6 py-4 font-bold text-slate-800 border-r border-slate-200">{item.area}</td>
                                                        <td className="px-6 py-4 text-right font-medium text-slate-400">{formatNumber(item.target)}</td>
                                                        <td className="px-6 py-4 text-right font-black text-slate-800">{formatNumber(item.sales)}</td>
                                                        <td className="px-6 py-4 text-center border-r border-slate-200">
                                                            <span className={`px-2 py-1 rounded-lg text-xs font-black ${salesSuccess ? 'text-emerald-600 bg-emerald-50' : 'text-rose-600 bg-rose-50'}`}>
                                                                {item.salesAch}%
                                                            </span>
                                                        </td>
                                                        <td className="px-6 py-4 text-right font-medium text-slate-400">{formatNumber(item.outTarget)}</td>
                                                        <td className="px-6 py-4 text-right font-black text-slate-800">{formatNumber(item.outActive)}</td>
                                                        <td className="px-6 py-4 text-center">
                                                            <span className={`px-2 py-1 rounded-lg text-xs font-black ${outSuccess ? 'text-emerald-600 bg-emerald-50' : 'text-amber-600 bg-amber-50'}`}>
                                                                {item.outAch}%
                                                            </span>
                                                        </td>
                                                    </tr>
                                                );
                                            }) : (
                                                <tr><td colSpan="9" className="px-6 py-8 text-center text-slate-400 font-bold">ไม่พบข้อมูล</td></tr>
                                            )}
                                        </tbody>
                                    </table>
                                </div>
                            </div>

                            {/* Topics Section */}
                            <div>
                                <h3 className="text-xl font-extrabold text-slate-800 mb-4 flex items-center gap-2">
                                    <Icons.Package size={24} className="text-indigo-500" />
                                    Sales by Product Topics
                                </h3>
                                <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
                                    <TopicCard 
                                        title="Product Push + Other Product" 
                                        sales={topicsAnalysis.pushOther.sales} 
                                        target={topicsAnalysis.pushOther.target}
                                        icon={<Icons.TrendingUp className="text-indigo-600"/>}
                                        bgClass="bg-indigo-100"
                                    />
                                    <TopicCard 
                                        title="Total Sales (Existing Product)" 
                                        sales={topicsAnalysis.existing.sales} 
                                        target={topicsAnalysis.existing.target}
                                        icon={<Icons.Award className="text-emerald-600"/>}
                                        bgClass="bg-emerald-100"
                                    />
                                    <TopicCard 
                                        title="New Product group" 
                                        sales={topicsAnalysis.newGroup.sales} 
                                        target={topicsAnalysis.newGroup.target}
                                        icon={<Icons.Target className="text-amber-600"/>}
                                        bgClass="bg-amber-100"
                                    />
                                </div>
                            </div>

                            {/* Charts Section */}
                            <div className="grid grid-cols-1 xl:grid-cols-2 gap-6">
                                {/* Trend Chart */}
                                <div className="glass-card p-6 rounded-3xl">
                                    <h3 className="text-lg font-extrabold text-slate-800 mb-6 flex items-center gap-2"><Icons.TrendingUp size={20} className="text-blue-500"/> Monthly Sales Trend</h3>
                                    <div className="h-80 w-full">
                                        <ResponsiveContainer width="100%" height="100%">
                                            <ComposedChart data={trendData} margin={{ top: 10, right: 10, left: 10, bottom: 5 }}>
                                                <CartesianGrid strokeDasharray="3 3" vertical={false} stroke="#f1f5f9"/>
                                                <XAxis dataKey="name" axisLine={false} tickLine={false} tick={{fill: '#64748b', fontWeight: 600, fontSize: 12}} />
                                                <YAxis axisLine={false} tickLine={false} tickFormatter={(val) => `${val/1000}k`} tick={{fill: '#64748b', fontWeight: 600, fontSize: 12}}/>
                                                <Tooltip content={<CustomTooltip />} cursor={{fill: '#f8fafc'}} />
                                                <Legend wrapperStyle={{paddingTop: '20px', fontWeight: 700}}/>
                                                <Bar dataKey="Target" fill="#cbd5e1" radius={[4, 4, 0, 0]} maxBarSize={40} />
                                                <Line type="monotone" dataKey="Sales" stroke="#3b82f6" strokeWidth={4} activeDot={{ r: 8, strokeWidth: 0 }} />
                                            </ComposedChart>
                                        </ResponsiveContainer>
                                    </div>
                                </div>

                                {/* Area Comparison Chart */}
                                <div className="glass-card p-6 rounded-3xl">
                                    <h3 className="text-lg font-extrabold text-slate-800 mb-6 flex items-center gap-2"><Icons.Target size={20} className="text-emerald-500"/> Performance by {selectedTeam === 'All' ? 'Team' : 'Area'}</h3>
                                    <div className="h-80 w-full">
                                        <ResponsiveContainer width="100%" height="100%">
                                            <BarChart data={areaChartData} margin={{ top: 10, right: 10, left: 10, bottom: 5 }}>
                                                <CartesianGrid strokeDasharray="3 3" vertical={false} stroke="#f1f5f9"/>
                                                <XAxis dataKey="name" axisLine={false} tickLine={false} tick={{fill: '#64748b', fontWeight: 600, fontSize: 12}} />
                                                <YAxis axisLine={false} tickLine={false} tickFormatter={(val) => `${val/1000000}M`} tick={{fill: '#64748b', fontWeight: 600, fontSize: 12}}/>
                                                <Tooltip content={<CustomTooltip />} cursor={{fill: '#f8fafc'}} />
                                                <Legend wrapperStyle={{paddingTop: '20px', fontWeight: 700}}/>
                                                <Bar dataKey="Target" fill="#cbd5e1" radius={[6, 6, 0, 0]} maxBarSize={40} />
                                                <Bar dataKey="Sales" fill="#10b981" radius={[6, 6, 0, 0]} maxBarSize={40} />
                                            </BarChart>
                                        </ResponsiveContainer>
                                    </div>
                                </div>
                            </div>

                            {/* Detailed Table */}
                            <div className="glass-card rounded-3xl overflow-hidden shadow-sm">
                                <div className="p-6 border-b border-slate-100 flex flex-col sm:flex-row justify-between items-start sm:items-center bg-white/50 gap-4">
                                    <h3 className="text-xl font-extrabold text-slate-800 flex items-center gap-3">
                                        <div className="bg-slate-100 p-2 rounded-lg"><Icons.Table size={20} className="text-slate-600"/></div>
                                        ตารางข้อมูลเจาะลึก (Product Level)
                                    </h3>
                                    <span className="text-sm font-bold text-slate-500 bg-white px-4 py-1.5 rounded-full shadow-sm border border-slate-100">
                                        แสดง {((currentPage - 1) * itemsPerPage) + 1} - {Math.min(currentPage * itemsPerPage, filteredData.length)} จาก {filteredData.length} รายการ
                                    </span>
                                </div>
                                
                                <div className="overflow-x-auto custom-scrollbar bg-white">
                                    <table className="w-full text-sm text-left text-slate-600">
                                        <thead className="text-xs text-slate-400 uppercase bg-slate-50/50 whitespace-nowrap">
                                            <tr>
                                                <th className="px-6 py-4 font-extrabold tracking-wider">Team</th>
                                                <th className="px-6 py-4 font-extrabold tracking-wider">Area</th>
                                                <th className="px-6 py-4 font-extrabold tracking-wider">Group Sales</th>
                                                <th className="px-6 py-4 font-extrabold tracking-wider">PRD Type</th>
                                                <th className="px-6 py-4 font-extrabold tracking-wider">Product Name</th>
                                                <th className="px-6 py-4 font-extrabold tracking-wider text-right">Target</th>
                                                <th className="px-6 py-4 font-extrabold tracking-wider text-right">Sales</th>
                                                <th className="px-6 py-4 font-extrabold tracking-wider text-center">% Ach</th>
                                            </tr>
                                        </thead>
                                        <tbody className="divide-y divide-slate-100">
                                            {paginatedData.length > 0 ? (
                                                paginatedData.map((item, index) => {
                                                    const achVal = calculateAch(item.sales, item.target);
                                                    const isSuccess = parseFloat(achVal) >= 100;
                                                    return (
                                                        <tr key={index} className="hover:bg-blue-50/50 transition-colors whitespace-nowrap">
                                                            <td className="px-6 py-4"><span className="bg-slate-100 text-slate-700 text-xs font-black px-3 py-1 rounded-lg">{item.team}</span></td>
                                                            <td className="px-6 py-4 font-bold text-slate-800">{item.salesCode}</td>
                                                            <td className="px-6 py-4 font-medium text-slate-500">{item.group}</td>
                                                            <td className="px-6 py-4 font-medium text-slate-500">{item.prdType}</td>
                                                            <td className="px-6 py-4 font-semibold text-slate-700">{item.productName}</td>
                                                            <td className="px-6 py-4 text-right text-slate-400 font-medium">{formatNumber(item.target)}</td>
                                                            <td className="px-6 py-4 text-right font-black text-slate-800">{formatTHB(item.sales)}</td>
                                                            <td className="px-6 py-4 text-center">
                                                                <span className={`px-3 py-1.5 rounded-xl text-xs font-black shadow-sm ${isSuccess ? 'bg-emerald-100 text-emerald-700 border border-emerald-200' : 'bg-rose-100 text-rose-700 border border-rose-200'}`}>
                                                                    {achVal}%
                                                                </span>
                                                            </td>
                                                        </tr>
                                                    );
                                                })
                                            ) : (
                                                <tr><td colSpan="8" className="px-6 py-12 text-center text-slate-400 font-bold text-lg">ไม่พบข้อมูลตามตัวกรอง</td></tr>
                                            )}
                                        </tbody>
                                    </table>
                                </div>

                                {/* Page Filter / Pagination */}
                                {filteredData.length > 0 && (
                                    <div className="flex flex-col sm:flex-row items-center justify-between p-5 bg-white border-t border-slate-100 gap-4">
                                        <button 
                                            onClick={() => setCurrentPage(p => Math.max(p - 1, 1))} 
                                            disabled={currentPage === 1} 
                                            className="w-full sm:w-auto flex items-center justify-center gap-2 px-5 py-2.5 text-sm font-bold text-slate-600 bg-white border border-slate-200 rounded-xl hover:bg-slate-50 hover:text-blue-600 disabled:opacity-40 shadow-sm transition-all"
                                        >
                                            <Icons.ChevronLeft size={18} /> ย้อนกลับ
                                        </button>
                                        
                                        <div className="flex items-center gap-3 bg-slate-50 px-4 py-2 rounded-xl border border-slate-200">
                                            <span className="text-sm font-bold text-slate-500 uppercase tracking-wide">เลือกหน้า</span>
                                            <select 
                                                value={currentPage} 
                                                onChange={(e) => setCurrentPage(Number(e.target.value))} 
                                                className="bg-white border border-slate-300 text-blue-700 text-base rounded-lg font-black block p-1.5 shadow-sm focus:ring-2 focus:ring-blue-500 outline-none cursor-pointer"
                                            >
                                                {Array.from({length: totalPages}, (_, i) => i + 1).map(page => (<option key={page} value={page}>{page}</option>))}
                                            </select>
                                            <span className="text-sm font-bold text-slate-500 uppercase tracking-wide">จาก {totalPages}</span>
                                        </div>

                                        <button 
                                            onClick={() => setCurrentPage(p => Math.min(p + 1, totalPages))} 
                                            disabled={currentPage === totalPages} 
                                            className="w-full sm:w-auto flex items-center justify-center gap-2 px-5 py-2.5 text-sm font-bold text-slate-600 bg-white border border-slate-200 rounded-xl hover:bg-slate-50 hover:text-blue-600 disabled:opacity-40 shadow-sm transition-all"
                                        >
                                            ถัดไป <Icons.ChevronRight size={18} />
                                        </button>
                                    </div>
                                )}
                            </div>
                        </div>

                    </main>
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<SalesDashboard />);
    </script>
</body>
</html>
