# Thoem-7.github.io
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <!-- ทำให้เป็นหน้าจอมือถือ และล็อกการซูม (App-like feel) -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    
    <!-- PWA / Apple Mobile Web App Meta Tags -->
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="Exec Dashboard">
    
    <!-- ไอคอนแอป (ถ้ามีรูปโลโก้บริษัท สามารถนำมาใส่แทนที่ลิงก์นี้ได้ครับ) -->
    <link rel="apple-touch-icon" href="https://cdn-icons-png.flaticon.com/512/8636/8636060.png">
    
    <title>Executive Sales Dashboard</title>
    
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
            width: 50px;
            height: 50px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        
        /* ป้องกันการเด้ง (Overscroll) บน iOS */
        html, body { overscroll-behavior-y: none; }
    </style>
</head>
<body class="text-slate-800">

    <div id="root"></div>

    <script type="text/babel">
        // ==========================================
        // 1. ตั้งค่า URL ของไฟล์ข้อมูลสำหรับดึงจาก GitHub
        // ==========================================
        const API_URLS = [
            'Executive Summary_OB2&HOR_26Y04_wk_02-2026.csv',
            'Sales_HOR_V03.csv',
            'Sales_OB2_V03.csv',
            'Target_HOR_V03.csv',
            'Target_OB2_V03.csv'
        ];

        const { useState, useMemo, useEffect } = React;
        const Recharts = window.Recharts;
        const { LineChart, Line, BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer, ComposedChart } = Recharts;

        // --- Inline Icons ---
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
            const [data, setData] = useState([]);
            const [outletData, setOutletData] = useState({});
            const [loadedFiles, setLoadedFiles] = useState([]);
            const [isLoading, setIsLoading] = useState(true); 
            const [loadingProgress, setLoadingProgress] = useState(0);
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

            // --- ฟังก์ชันนำข้อมูลมาประมวลผล ---
            const processFilesData = (fileResults) => {
                let consolidated = {};
                let newOutlet = {}; 

                fileResults.forEach(({ name, text }) => {
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
                        
                        const idxSales = headers.findIndex(h => h.toLowerCase() === 'sales');
                        const idxTarget = headers.findIndex(h => h.toLowerCase() === 'target');
                        const idxAmount = headers.findIndex(h => h.toLowerCase() === 'amount');
                        const idxType = headers.findIndex(h => h.toLowerCase() === 'type');

                        for(let i=1; i<lines.length; i++) {
                            const row = splitCSV(lines[i]);
                            if (row.length === headers.length) {
                                let salesVal = 0, targetVal = 0;
                                if (idxSales >= 0) salesVal = cleanNumber(row[idxSales]);
                                if (idxTarget >= 0) targetVal = cleanNumber(row[idxTarget]);

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
                setCurrentPage(1);
            };

            // --- ฟังก์ชันดึงข้อมูลอัตโนมัติจาก GitHub (API Auto-Fetch) ---
            const fetchAPIData = async () => {
                setIsLoading(true);
                setApiError(false);
                setLoadingProgress(0);
                try {
                    let fileResults = [];
                    for(let i=0; i<API_URLS.length; i++) {
                        // ใช้ encodeURI ป้องกัน Error กรณีชื่อไฟล์มีการเว้นวรรค
                        const url = encodeURI(API_URLS[i]);
                        const res = await fetch(url, { cache: "no-store" });
                        if (!res.ok) throw new Error(`HTTP Error ${res.status} on file ${API_URLS[i]}`);
                        const text = await res.text();
                        fileResults.push({ name: API_URLS[i], text: text });
                        setLoadingProgress(Math.round(((i + 1) / API_URLS.length) * 100));
                    }
                    processFilesData(fileResults);
                    setLoadedFiles(API_URLS);
                } catch (err) {
                    console.log("Auto-fetch API ไม่สำเร็จ:", err);
                    setApiError(true);
                } finally {
                    setIsLoading(false);
                }
            };

            // โหลดข้อมูลอัตโนมัติเมื่อเปิดแอป
            useEffect(() => { fetchAPIData(); }, []);

            // --- ปุ่มอัปโหลดสำรอง (Manual Upload) ---
            const handleFileUpload = async (event) => {
                const files = Array.from(event.target.files);
                if (files.length === 0) return;
                setIsLoading(true);
                
                let fileNames = [];
                const readAsText = (file) => new Promise((resolve) => {
                    fileNames.push(file.name);
                    const reader = new FileReader();
                    reader.onload = (e) => resolve({ name: file.name, text: e.target.result });
                    reader.readAsText(file);
                });

                const fileResults = await Promise.all(files.map(readAsText));
                processFilesData(fileResults);
                setLoadedFiles(fileNames);
                setIsLoading(false);
                setApiError(false);
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
                filteredData.forEach(d => { totalSales += d.sales; totalTarget += d.target; });

                let uniqueSalesCodes = [...new Set(filteredData.map(d => d.salesCode))];
                let outTarget = 0, outActive = 0;
                uniqueSalesCodes.forEach(code => {
                    if (outletData[code]) {
                        outTarget += outletData[code].target;
                        outActive += outletData[code].active;
                    }
                });

                return { 
                    totalSales, totalTarget, salesAch: calculateAch(totalSales, totalTarget),
                    outTarget, outActive, outAch: calculateAch(outActive, outTarget)
                };
            }, [filteredData, outletData]);

            // Topics Analysis
            const topicsAnalysis = useMemo(() => {
                const res = { pushOther: { sales: 0, target: 0 }, existing: { sales: 0, target: 0 }, newGroup: { sales: 0, target: 0 } };
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
                    if (!grouped[key]) grouped[key] = { team: d.team, sup: d.sup, area: d.salesCode, sales: 0, target: 0 };
                    grouped[key].sales += d.sales;
                    grouped[key].target += d.target;
                });

                return Object.values(grouped).map(g => {
                    const out = outletData[g.area] || { target: 0, active: 0 };
                    return {
                        ...g, salesAch: calculateAch(g.sales, g.target),
                        outTarget: out.target, outActive: out.active, outAch: calculateAch(out.active, out.target)
                    };
                }).sort((a, b) => a.team.localeCompare(b.team) || a.area.localeCompare(b.area));
            }, [filteredData, outletData]);

            // Area Comparison Chart
            const areaChartData = useMemo(() => {
                const grouped = {};
                filteredData.forEach(d => {
                    let key = selectedTeam === 'All' ? d.team : d.salesCode;
                    if (!grouped[key]) grouped[key] = { name: key, Sales: 0, Target: 0 };
                    grouped[key].Sales += d.sales; grouped[key].Target += d.target;
                });
                return Object.values(grouped).map(d => ({ ...d, Ach: calculateAch(d.Sales, d.Target) })).sort((a, b) => b.Sales - a.Sales);
            }, [filteredData, selectedTeam]);

            // Monthly Trend Chart
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

            React.useEffect(() => { setCurrentPage(1); }, [selectedTeam, selectedArea, selectedYear, selectedQuarter, selectedMonth, selectedSupCode, selectedGroupSales, selectedPrdType, selectedProductName]);

            // --- หน้าจอดาวน์โหลดแอป ---
            if (isLoading) {
                return (
                    <div className="flex flex-col items-center justify-center min-h-screen bg-slate-900 text-white">
                        <div className="loader mb-6"></div>
                        <h2 className="text-2xl font-extrabold mb-2">กำลังซิงค์ข้อมูล...</h2>
                        <p className="text-slate-400 font-medium">{loadingProgress}% ดึงข้อมูลล่าสุดจากระบบ</p>
                    </div>
                );
            }

            // --- หน้าจอ Fallback (ถ้าเปิด Offline หรือไม่มีไฟล์บน GitHub) ---
            if (data.length === 0) {
                return (
                    <div className="flex flex-col items-center justify-center min-h-screen bg-slate-900 text-white p-6 font-sans">
                        <div className="bg-slate-800 p-10 rounded-[2rem] shadow-2xl text-center max-w-lg w-full border border-slate-700">
                            <div className="bg-gradient-to-tr from-blue-500 to-indigo-400 w-24 h-24 rounded-3xl flex items-center justify-center mx-auto mb-8 shadow-lg shadow-blue-500/50 transform rotate-3">
                                <Icons.Upload className="text-white" size={40} />
                            </div>
                            <h1 className="text-3xl font-extrabold mb-3 tracking-tight">Executive App</h1>
                            <p className="text-slate-400 mb-8 font-medium">รอการเชื่อมต่อข้อมูล...</p>
                            
                            {apiError && (
                                <div className="bg-rose-500/10 text-rose-300 p-4 rounded-2xl mb-8 text-sm font-semibold border border-rose-500/30 text-left">
                                    <p className="flex items-center gap-2"><Icons.Filter size={18}/> ไม่พบไฟล์ข้อมูลอัตโนมัติบน GitHub</p>
                                    <p className="text-xs font-normal mt-2 opacity-80">คุณอาจจะยังไม่ได้อัปโหลดไฟล์ CSV ทั้ง 5 ไฟล์ไว้ใน GitHub Repository หรือชื่อไฟล์อาจจะไม่ตรงกัน กดนำเข้าเองเพื่อทดลองดูได้ครับ</p>
                                </div>
                            )}

                            <label className="cursor-pointer bg-blue-600 hover:bg-blue-500 text-white font-bold py-4 px-8 rounded-2xl transition-all duration-300 flex items-center justify-center gap-3 shadow-xl transform active:scale-95 w-full">
                                <Icons.Upload size={22} />
                                <span className="text-lg">นำเข้าไฟล์ (Manual)</span>
                                <input type="file" accept=".csv" multiple onChange={handleFileUpload} className="hidden" />
                            </label>
                        </div>
                    </div>
                );
            }

            // --- Component ของ Topic Card ---
            const TopicCard = ({ title, sales, target, icon, bgClass }) => {
                const ach = calculateAch(sales, target);
                const isSuccess = parseFloat(ach) >= 100;
                return (
                    <div className="glass-card p-5 rounded-3xl relative overflow-hidden flex flex-col justify-between h-full bg-white">
                        <div className="flex justify-between items-start mb-4">
                            <div>
                                <h4 className="text-xs font-black text-slate-400 uppercase tracking-widest mb-2">{title}</h4>
                                <div className="text-2xl font-black text-slate-800">{formatTHB(sales)}</div>
                                <p className="text-xs font-bold text-slate-400 mt-1">Target: {formatTHB(target)}</p>
                            </div>
                            <div className={`${bgClass} p-3.5 rounded-2xl shadow-inner`}>{icon}</div>
                        </div>
                        
                        <div className="mt-2 pt-4 border-t border-slate-100 flex items-center justify-between">
                            <span className="text-xs font-bold text-slate-500">Achievement</span>
                            <span className={`px-3 py-1 rounded-xl text-xs font-black ${isSuccess ? 'bg-emerald-100 text-emerald-700' : 'bg-rose-100 text-rose-700'}`}>
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
                    {/* Header (App Bar) */}
                    <nav className="bg-slate-900 shadow-xl sticky top-0 z-40">
                        <div className="max-w-[1600px] mx-auto px-4 sm:px-6 lg:px-8">
                            <div className="flex justify-between h-16 sm:h-20 items-center">
                                <div className="flex items-center gap-3 sm:gap-4">
                                    <div className="bg-gradient-to-br from-blue-500 to-indigo-600 p-2 sm:p-2.5 rounded-xl shadow-inner">
                                        <Icons.Award className="text-white w-5 h-5 sm:w-7 sm:h-7" />
                                    </div>
                                    <div>
                                        <h1 className="text-base sm:text-2xl font-extrabold text-white tracking-wide leading-tight">Sales Dashboard</h1>
                                        <p className="text-[10px] sm:text-xs text-blue-200 font-medium">OB2 & HOR Summary</p>
                                    </div>
                                </div>
                                <div className="flex gap-2">
                                    <button onClick={fetchAPIData} className="text-white bg-white/10 hover:bg-white/20 p-2.5 rounded-xl transition-all flex items-center gap-2">
                                        <Icons.Refresh size={20} />
                                        <span className="hidden md:inline font-bold text-sm">อัปเดตข้อมูล</span>
                                    </button>
                                    <label className="cursor-pointer text-slate-900 bg-white hover:bg-blue-50 p-2.5 rounded-xl transition-all flex items-center shadow-sm">
                                        <Icons.Upload size={20} />
                                        <input type="file" accept=".csv" multiple onChange={handleFileUpload} className="hidden" />
                                    </label>
                                </div>
                            </div>
                        </div>
                    </nav>

                    {/* Main Layout Area */}
                    <main className="max-w-[1600px] w-full mx-auto px-4 sm:px-6 lg:px-8 py-6 sm:py-8 flex flex-col xl:flex-row gap-6 sm:gap-8 items-start flex-1">
                        
                        {/* Sidebar: Filters Section (ซ้ายมือ) */}
                        <aside className="w-full xl:w-80 flex-shrink-0">
                            <div className="glass-card p-5 sm:p-6 rounded-3xl xl:sticky xl:top-28">
                                <div className="flex items-center gap-3 mb-5 pb-4 border-b border-slate-100">
                                    <div className="bg-blue-100 p-2.5 rounded-xl"><Icons.Filter size={20} className="text-blue-700"/></div>
                                    <span className="font-extrabold text-slate-800 uppercase tracking-wide text-lg">Filters</span>
                                </div>
                                
                                <div className="grid grid-cols-2 xl:grid-cols-1 gap-4">
                                    <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                                        <label className="block text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1.5 ml-1">Year (ปี)</label>
                                        <select value={selectedYear} onChange={(e) => setSelectedYear(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 block p-3 font-bold shadow-sm appearance-none">
                                            <option value="All">ทุกปี (All)</option>
                                            {filterOptions.years.map(y => <option key={y} value={y}>{y}</option>)}
                                        </select>
                                    </div>
                                    <div>
                                        <label className="block text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1.5 ml-1">Quarter</label>
                                        <select value={selectedQuarter} onChange={(e) => setSelectedQuarter(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 block p-3 font-bold shadow-sm appearance-none">
                                            <option value="All">ทุกไตรมาส</option>
                                            {filterOptions.quarters.map(q => <option key={q} value={q}>{q}</option>)}
                                        </select>
                                    </div>
                                    <div>
                                        <label className="block text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1.5 ml-1">Month</label>
                                        <select value={selectedMonth} onChange={(e) => setSelectedMonth(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 block p-3 font-bold shadow-sm appearance-none">
                                            <option value="All">ทุกเดือน</option>
                                            {MONTH_NAMES_TH.map((m, index) => <option key={index} value={index+1}>{m}</option>)}
                                        </select>
                                    </div>

                                    <div className="col-span-2 xl:col-span-1 border-t border-slate-100 pt-3 mt-1"></div>

                                    <div>
                                        <label className="block text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1.5 ml-1">Team</label>
                                        <select value={selectedTeam} onChange={(e) => setSelectedTeam(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 block p-3 font-bold shadow-sm appearance-none">
                                            <option value="All">ทุกทีม</option>
                                            {filterOptions.teams.map(t => <option key={t} value={t}>{t}</option>)}
                                        </select>
                                    </div>
                                    <div>
                                        <label className="block text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1.5 ml-1">Area</label>
                                        <select value={selectedArea} onChange={(e) => setSelectedArea(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 block p-3 font-bold shadow-sm appearance-none">
                                            <option value="All">ทุกเขต</option>
                                            {filterOptions.areas.map(a => <option key={a} value={a}>{a}</option>)}
                                        </select>
                                    </div>
                                    <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                                        <label className="block text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1.5 ml-1">SUP Code</label>
                                        <select value={selectedSupCode} onChange={(e) => setSelectedSupCode(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 block p-3 font-bold shadow-sm appearance-none">
                                            <option value="All">ทุก SUP</option>
                                            {filterOptions.supCodes.map(s => <option key={s} value={s}>{s}</option>)}
                                        </select>
                                    </div>

                                    <div className="col-span-2 xl:col-span-1 border-t border-slate-100 pt-3 mt-1"></div>

                                    <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                                        <label className="block text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1.5 ml-1">Group Sales</label>
                                        <select value={selectedGroupSales} onChange={(e) => setSelectedGroupSales(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 block p-3 font-bold shadow-sm appearance-none">
                                            <option value="All">ทุก Group</option>
                                            {filterOptions.groupSales.map(c => <option key={c} value={c}>{c}</option>)}
                                        </select>
                                    </div>
                                    <div className="col-span-2 sm:col-span-1 xl:col-span-1">
                                        <label className="block text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1.5 ml-1">PRD Type</label>
                                        <select value={selectedPrdType} onChange={(e) => setSelectedPrdType(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 block p-3 font-bold shadow-sm appearance-none">
                                            <option value="All">ทุก PRD Type</option>
                                            {filterOptions.prdTypes.map(c => <option key={c} value={c}>{c}</option>)}
                                        </select>
                                    </div>
                                    <div className="col-span-2 xl:col-span-1">
                                        <label className="block text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1.5 ml-1">Product Name</label>
                                        <select value={selectedProductName} onChange={(e) => setSelectedProductName(e.target.value)} className="w-full bg-slate-50 border border-slate-200 text-slate-800 text-sm rounded-xl focus:ring-2 focus:ring-blue-500 block p-3 font-bold shadow-sm appearance-none">
                                            <option value="All">ทุกสินค้า</option>
                                            {filterOptions.productNames.map(c => <option key={c} value={c}>{c}</option>)}
                                        </select>
                                    </div>
                                </div>
                            </div>
                        </aside>

                        {/* Right Area: Content & Data */}
                        <div className="flex-1 space-y-6 sm:space-y-8 min-w-0 w-full">
                            
                            {/* Master KPIs */}
                            <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                                {/* Sales KPI */}
                                <div className="glass-card p-6 sm:p-8 rounded-[2rem] flex items-center justify-between relative overflow-hidden bg-gradient-to-br from-blue-600 to-indigo-700 text-white shadow-xl shadow-blue-500/20">
                                    <div className="z-10 w-full">
                                        <p className="text-blue-200 text-xs sm:text-sm font-extrabold uppercase tracking-widest mb-2">Target vs Sales</p>
                                        <div className="flex flex-col sm:flex-row sm:items-baseline gap-1 sm:gap-3 mb-4">
                                            <h3 className="text-4xl sm:text-5xl font-black">{formatTHB(kpis.totalSales)}</h3>
                                            <span className="text-lg font-bold text-blue-300">/ {formatTHB(kpis.totalTarget)}</span>
                                        </div>
                                        <div className="inline-flex items-center gap-3 bg-white/10 px-4 py-2 rounded-xl backdrop-blur-md border border-white/20">
                                            <span className="text-xs font-bold uppercase text-blue-100">Achieved</span>
                                            <span className={`text-xl font-black ${kpis.salesAch >= 100 ? 'text-emerald-300' : 'text-rose-300'}`}>{kpis.salesAch}%</span>
                                        </div>
                                    </div>
                                    <Icons.Dollar size={120} className="text-white opacity-10 absolute -right-6 -bottom-6 transform -rotate-12" />
                                </div>

                                {/* Outlet KPI */}
                                <div className="glass-card p-6 sm:p-8 rounded-[2rem] flex items-center justify-between relative overflow-hidden bg-gradient-to-br from-amber-500 to-orange-600 text-white shadow-xl shadow-amber-500/20">
                                    <div className="z-10 w-full">
                                        <p className="text-amber-200 text-xs sm:text-sm font-extrabold uppercase tracking-widest mb-2">Outlet Target vs Active</p>
                                        <div className="flex flex-col sm:flex-row sm:items-baseline gap-1 sm:gap-3 mb-4">
                                            <h3 className="text-4xl sm:text-5xl font-black">{formatNumber(kpis.outActive)}</h3>
                                            <span className="text-lg font-bold text-amber-200">/ {formatNumber(kpis.outTarget)}</span>
                                        </div>
                                        <div className="inline-flex items-center gap-3 bg-white/10 px-4 py-2 rounded-xl backdrop-blur-md border border-white/20">
                                            <span className="text-xs font-bold uppercase text-amber-100">Achieved</span>
                                            <span className={`text-xl font-black ${kpis.outAch >= 100 ? 'text-emerald-300' : 'text-rose-200'}`}>{kpis.outAch}%</span>
                                        </div>
                                    </div>
                                    <Icons.Store size={120} className="text-white opacity-10 absolute -right-6 -bottom-6 transform rotate-12" />
                                </div>
                            </div>

                            {/* Executive Summary Table */}
                            <div className="glass-card rounded-[2rem] overflow-hidden shadow-sm bg-white">
                                <div className="p-5 sm:p-6 border-b border-slate-100 flex items-center gap-3 bg-slate-50/50">
                                    <div className="bg-indigo-100 p-2.5 rounded-xl"><Icons.Table size={20} className="text-indigo-600"/></div>
                                    <h3 className="text-lg sm:text-xl font-extrabold text-slate-800">Executive Summary <span className="hidden sm:inline">By Team / Area</span></h3>
                                </div>
                                <div className="overflow-x-auto custom-scrollbar">
                                    <table className="w-full text-sm text-left text-slate-600">
                                        <thead className="text-[10px] sm:text-xs text-slate-500 uppercase bg-slate-50 whitespace-nowrap">
                                            <tr>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider border-r border-slate-100">Team</th>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider border-r border-slate-100">SUP Code</th>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider border-r border-slate-200">Area</th>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider text-right text-blue-600">Target</th>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider text-right text-blue-600">Sales</th>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider text-center border-r border-slate-200 text-blue-600">% Ach</th>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider text-right text-amber-600">Out. Target</th>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider text-right text-amber-600">Out. Active</th>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider text-center text-amber-600">% Ach</th>
                                            </tr>
                                        </thead>
                                        <tbody className="divide-y divide-slate-100">
                                            {execSummaryData.length > 0 ? execSummaryData.map((item, idx) => {
                                                const salesSuccess = parseFloat(item.salesAch) >= 100;
                                                const outSuccess = parseFloat(item.outAch) >= 100;
                                                return (
                                                    <tr key={idx} className="hover:bg-slate-50 transition-colors whitespace-nowrap">
                                                        <td className="px-4 sm:px-6 py-4 font-black text-slate-700 border-r border-slate-100">{item.team}</td>
                                                        <td className="px-4 sm:px-6 py-4 font-medium text-slate-500 border-r border-slate-100">{item.sup}</td>
                                                        <td className="px-4 sm:px-6 py-4 font-bold text-slate-800 border-r border-slate-200">{item.area}</td>
                                                        <td className="px-4 sm:px-6 py-4 text-right font-medium text-slate-400">{formatNumber(item.target)}</td>
                                                        <td className="px-4 sm:px-6 py-4 text-right font-black text-slate-800">{formatNumber(item.sales)}</td>
                                                        <td className="px-4 sm:px-6 py-4 text-center border-r border-slate-200">
                                                            <span className={`px-2.5 py-1 rounded-lg text-xs font-black ${salesSuccess ? 'text-emerald-600 bg-emerald-100' : 'text-rose-600 bg-rose-100'}`}>
                                                                {item.salesAch}%
                                                            </span>
                                                        </td>
                                                        <td className="px-4 sm:px-6 py-4 text-right font-medium text-slate-400">{formatNumber(item.outTarget)}</td>
                                                        <td className="px-4 sm:px-6 py-4 text-right font-black text-slate-800">{formatNumber(item.outActive)}</td>
                                                        <td className="px-4 sm:px-6 py-4 text-center">
                                                            <span className={`px-2.5 py-1 rounded-lg text-xs font-black ${outSuccess ? 'text-emerald-600 bg-emerald-100' : 'text-amber-600 bg-amber-100'}`}>
                                                                {item.outAch}%
                                                            </span>
                                                        </td>
                                                    </tr>
                                                );
                                            }) : (
                                                <tr><td colSpan="9" className="px-6 py-10 text-center text-slate-400 font-bold">ไม่พบข้อมูล</td></tr>
                                            )}
                                        </tbody>
                                    </table>
                                </div>
                            </div>

                            {/* Topics Section */}
                            <div>
                                <h3 className="text-xl font-extrabold text-slate-800 mb-4 flex items-center gap-2 pl-2">
                                    <Icons.Package size={24} className="text-indigo-500" />
                                    Sales by Product Topics
                                </h3>
                                <div className="grid grid-cols-1 md:grid-cols-3 gap-4 sm:gap-6">
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
                                <div className="glass-card p-4 sm:p-6 rounded-[2rem] bg-white">
                                    <h3 className="text-lg font-extrabold text-slate-800 mb-6 flex items-center gap-2"><Icons.TrendingUp size={20} className="text-blue-500"/> Monthly Sales Trend</h3>
                                    <div className="h-64 sm:h-80 w-full">
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
                                <div className="glass-card p-4 sm:p-6 rounded-[2rem] bg-white">
                                    <h3 className="text-lg font-extrabold text-slate-800 mb-6 flex items-center gap-2"><Icons.Target size={20} className="text-emerald-500"/> Performance by {selectedTeam === 'All' ? 'Team' : 'Area'}</h3>
                                    <div className="h-64 sm:h-80 w-full">
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

                            {/* Detailed Table (Product Level) */}
                            <div className="glass-card rounded-[2rem] overflow-hidden shadow-sm bg-white">
                                <div className="p-5 sm:p-6 border-b border-slate-100 flex flex-col sm:flex-row justify-between items-start sm:items-center bg-slate-50/50 gap-4">
                                    <h3 className="text-lg sm:text-xl font-extrabold text-slate-800 flex items-center gap-3">
                                        <div className="bg-slate-100 p-2 rounded-lg"><Icons.Table size={20} className="text-slate-600"/></div>
                                        ตารางข้อมูลเจาะลึก (Product)
                                    </h3>
                                    <span className="text-xs font-bold text-slate-500 bg-white px-4 py-2 rounded-full shadow-sm border border-slate-100">
                                        แสดง {((currentPage - 1) * itemsPerPage) + 1} - {Math.min(currentPage * itemsPerPage, filteredData.length)} จาก {filteredData.length}
                                    </span>
                                </div>
                                
                                <div className="overflow-x-auto custom-scrollbar">
                                    <table className="w-full text-sm text-left text-slate-600">
                                        <thead className="text-[10px] sm:text-xs text-slate-400 uppercase bg-slate-50 whitespace-nowrap">
                                            <tr>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider">Team</th>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider">Area</th>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider">Group Sales</th>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider">PRD Type</th>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider min-w-[150px]">Product Name</th>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider text-right">Target</th>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider text-right">Sales</th>
                                                <th className="px-4 sm:px-6 py-4 font-extrabold tracking-wider text-center">% Ach</th>
                                            </tr>
                                        </thead>
                                        <tbody className="divide-y divide-slate-100">
                                            {paginatedData.length > 0 ? (
                                                paginatedData.map((item, index) => {
                                                    const achVal = calculateAch(item.sales, item.target);
                                                    const isSuccess = parseFloat(achVal) >= 100;
                                                    return (
                                                        <tr key={index} className="hover:bg-blue-50/50 transition-colors whitespace-nowrap">
                                                            <td className="px-4 sm:px-6 py-4"><span className="bg-slate-100 text-slate-700 text-[10px] sm:text-xs font-black px-3 py-1.5 rounded-lg">{item.team}</span></td>
                                                            <td className="px-4 sm:px-6 py-4 font-bold text-slate-800">{item.salesCode}</td>
                                                            <td className="px-4 sm:px-6 py-4 font-medium text-slate-500">{item.group}</td>
                                                            <td className="px-4 sm:px-6 py-4 font-medium text-slate-500">{item.prdType}</td>
                                                            <td className="px-4 sm:px-6 py-4 font-semibold text-slate-700 truncate max-w-[200px]">{item.productName}</td>
                                                            <td className="px-4 sm:px-6 py-4 text-right text-slate-400 font-medium">{formatNumber(item.target)}</td>
                                                            <td className="px-4 sm:px-6 py-4 text-right font-black text-slate-800">{formatTHB(item.sales)}</td>
                                                            <td className="px-4 sm:px-6 py-4 text-center">
                                                                <span className={`px-2.5 py-1.5 rounded-xl text-[10px] sm:text-xs font-black shadow-sm ${isSuccess ? 'bg-emerald-100 text-emerald-700 border border-emerald-200' : 'bg-rose-100 text-rose-700 border border-rose-200'}`}>
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
                                    <div className="flex flex-col sm:flex-row items-center justify-between p-4 sm:p-5 bg-slate-50 border-t border-slate-100 gap-4">
                                        <button 
                                            onClick={() => setCurrentPage(p => Math.max(p - 1, 1))} 
                                            disabled={currentPage === 1} 
                                            className="w-full sm:w-auto flex items-center justify-center gap-2 px-5 py-3 sm:py-2.5 text-sm font-bold text-slate-600 bg-white border border-slate-200 rounded-xl hover:bg-slate-50 hover:text-blue-600 disabled:opacity-40 shadow-sm transition-all"
                                        >
                                            <Icons.ChevronLeft size={18} /> ย้อนกลับ
                                        </button>
                                        
                                        <div className="flex items-center justify-center gap-3 bg-white px-4 py-2 rounded-xl border border-slate-200 w-full sm:w-auto shadow-sm">
                                            <span className="text-xs font-bold text-slate-500 uppercase tracking-wide">หน้า</span>
                                            <select 
                                                value={currentPage} 
                                                onChange={(e) => setCurrentPage(Number(e.target.value))} 
                                                className="bg-slate-50 border border-slate-200 text-blue-700 text-base rounded-lg font-black block p-1.5 focus:ring-2 focus:ring-blue-500 outline-none cursor-pointer"
                                            >
                                                {Array.from({length: totalPages}, (_, i) => i + 1).map(page => (<option key={page} value={page}>{page}</option>))}
                                            </select>
                                            <span className="text-xs font-bold text-slate-500 uppercase tracking-wide">จาก {totalPages}</span>
                                        </div>

                                        <button 
                                            onClick={() => setCurrentPage(p => Math.min(p + 1, totalPages))} 
                                            disabled={currentPage === totalPages} 
                                            className="w-full sm:w-auto flex items-center justify-center gap-2 px-5 py-3 sm:py-2.5 text-sm font-bold text-slate-600 bg-white border border-slate-200 rounded-xl hover:bg-slate-50 hover:text-blue-600 disabled:opacity-40 shadow-sm transition-all"
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
