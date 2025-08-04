<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Comparador PDF vs Excel</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }
        
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: white;
            border-radius: 20px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            overflow: hidden;
        }
        
        .header {
            background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
            color: white;
            padding: 30px;
            text-align: center;
        }
        
        .header h1 {
            font-size: 2.5rem;
            margin-bottom: 10px;
            font-weight: 700;
        }
        
        .header p {
            font-size: 1.1rem;
            opacity: 0.9;
        }
        
        .upload-section {
            padding: 40px;
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 30px;
        }
        
        .upload-box {
            border: 3px dashed #e0e7ff;
            border-radius: 15px;
            padding: 30px;
            text-align: center;
            transition: all 0.3s ease;
            background: #f8faff;
        }
        
        .upload-box:hover {
            border-color: #4facfe;
            background: #f0f9ff;
            transform: translateY(-2px);
        }
        
        .upload-box.dragover {
            border-color: #4facfe;
            background: #e0f2fe;
        }
        
        .upload-icon {
            font-size: 3rem;
            margin-bottom: 15px;
            color: #4facfe;
        }
        
        .upload-box h3 {
            color: #1e293b;
            margin-bottom: 10px;
            font-size: 1.3rem;
        }
        
        .upload-box p {
            color: #64748b;
            margin-bottom: 20px;
        }
        
        .file-input {
            display: none;
        }
        
        .upload-btn {
            background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 25px;
            cursor: pointer;
            font-weight: 600;
            transition: all 0.3s ease;
        }
        
        .upload-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 20px rgba(79, 172, 254, 0.3);
        }
        
        .file-info {
            margin-top: 15px;
            padding: 10px;
            background: #e0f2fe;
            border-radius: 8px;
            color: #0369a1;
            font-weight: 500;
        }
        
        .compare-section {
            padding: 0 40px 40px;
            text-align: center;
        }
        
        .compare-btn {
            background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
            color: white;
            border: none;
            padding: 15px 40px;
            border-radius: 30px;
            font-size: 1.1rem;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
            disabled: opacity 0.5;
        }
        
        .compare-btn:hover:not(:disabled) {
            transform: translateY(-2px);
            box-shadow: 0 15px 30px rgba(245, 87, 108, 0.3);
        }
        
        .compare-btn:disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }
        
        .results-section {
            padding: 40px;
            background: #f8faff;
            display: none;
        }
        
        .results-header {
            text-align: center;
            margin-bottom: 30px;
        }
        
        .results-header h2 {
            color: #1e293b;
            font-size: 2rem;
            margin-bottom: 10px;
        }
        
        .stats {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }
        
        .stat-card {
            background: white;
            padding: 20px;
            border-radius: 15px;
            text-align: center;
            box-shadow: 0 5px 15px rgba(0,0,0,0.1);
        }
        
        .stat-number {
            font-size: 2rem;
            font-weight: 700;
            margin-bottom: 5px;
        }
        
        .stat-number.matches { color: #10b981; }
        .stat-number.discrepancies { color: #ef4444; }
        .stat-number.missing { color: #f59e0b; }
        
        .stat-label {
            color: #64748b;
            font-weight: 500;
        }
        
        .discrepancies-list {
            background: white;
            border-radius: 15px;
            padding: 30px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.1);
        }
        
        .discrepancy-item {
            border-left: 4px solid #ef4444;
            padding: 20px;
            margin-bottom: 20px;
            background: #fef2f2;
            border-radius: 0 10px 10px 0;
        }
        
        .discrepancy-sku {
            font-weight: 700;
            color: #dc2626;
            font-size: 1.1rem;
            margin-bottom: 10px;
        }
        
        .discrepancy-details {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
        }
        
        .detail-column h4 {
            color: #374151;
            margin-bottom: 8px;
            font-size: 0.9rem;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }
        
        .detail-item {
            background: white;
            padding: 8px 12px;
            border-radius: 6px;
            margin-bottom: 5px;
            font-size: 0.9rem;
        }
        
        .loading {
            text-align: center;
            padding: 40px;
            color: #64748b;
        }
        
        .loading-spinner {
            width: 40px;
            height: 40px;
            border: 4px solid #e0e7ff;
            border-top: 4px solid #4facfe;
            border-radius: 50%;
            animation: spin 1s linear infinite;
            margin: 0 auto 20px;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        @media (max-width: 768px) {
            .upload-section {
                grid-template-columns: 1fr;
                padding: 20px;
            }
            
            .header h1 {
                font-size: 2rem;
            }
            
            .stats {
                grid-template-columns: 1fr;
            }
            
            .discrepancy-details {
                grid-template-columns: 1fr;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>📊 Comparador PDF vs Excel</h1>
            <p>Encuentra discrepancias entre tu PDF y base de datos Excel</p>
        </div>
        
        <div class="upload-section">
            <div class="upload-box" id="pdfUpload">
                <div class="upload-icon">📄</div>
                <h3>Subir PDF</h3>
                <p>Arrastra tu archivo PDF aquí o haz clic para seleccionar</p>
                <button class="upload-btn" onclick="document.getElementById('pdfFile').click()">
                    Seleccionar PDF
                </button>
                <input type="file" id="pdfFile" class="file-input" accept=".pdf">
                <div id="pdfInfo" class="file-info" style="display: none;"></div>
            </div>
            
            <div class="upload-box" id="excelUpload">
                <div class="upload-icon">📊</div>
                <h3>Subir Excel</h3>
                <p>Arrastra tu archivo Excel aquí o haz clic para seleccionar</p>
                <button class="upload-btn" onclick="document.getElementById('excelFile').click()">
                    Seleccionar Excel
                </button>
                <input type="file" id="excelFile" class="file-input" accept=".xlsx,.xls">
                <div id="excelInfo" class="file-info" style="display: none;"></div>
            </div>
        </div>
        
        <div class="compare-section">
            <button class="compare-btn" id="compareBtn" disabled onclick="compareFiles()">
                🔍 Comparar Archivos
            </button>
        </div>
        
        <div class="results-section" id="resultsSection">
            <div class="loading" id="loadingDiv">
                <div class="loading-spinner"></div>
                <p>Analizando archivos y buscando discrepancias...</p>
            </div>
            
            <div id="resultsContent" style="display: none;">
                <div class="results-header">
                    <h2>📋 Resultados del Análisis</h2>
                </div>
                
                <div class="stats">
                    <div class="stat-card">
                        <div class="stat-number matches" id="matchesCount">0</div>
                        <div class="stat-label">Coincidencias</div>
                    </div>
                    <div class="stat-card">
                        <div class="stat-number discrepancies" id="discrepanciesCount">0</div>
                        <div class="stat-label">Discrepancias</div>
                    </div>
                    <div class="stat-card">
                        <div class="stat-number missing" id="missingCount">0</div>
                        <div class="stat-label">No encontrados</div>
                    </div>
                </div>
                
                <div class="discrepancies-list" id="discrepanciesList">
                    <!-- Los resultados se mostrarán aquí -->
                </div>
            </div>
        </div>
    </div>

    <script>
        let pdfData = null;
        let excelData = null;
        
        // Configurar PDF.js
        pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.worker.min.js';
        
        // Configurar drag and drop
        setupDragAndDrop('pdfUpload', 'pdfFile');
        setupDragAndDrop('excelUpload', 'excelFile');
        
        // Event listeners para archivos
        document.getElementById('pdfFile').addEventListener('change', handlePdfFile);
        document.getElementById('excelFile').addEventListener('change', handleExcelFile);
        
        function setupDragAndDrop(uploadBoxId, fileInputId) {
            const uploadBox = document.getElementById(uploadBoxId);
            const fileInput = document.getElementById(fileInputId);
            
            uploadBox.addEventListener('dragover', (e) => {
                e.preventDefault();
                uploadBox.classList.add('dragover');
            });
            
            uploadBox.addEventListener('dragleave', () => {
                uploadBox.classList.remove('dragover');
            });
            
            uploadBox.addEventListener('drop', (e) => {
                e.preventDefault();
                uploadBox.classList.remove('dragover');
                const files = e.dataTransfer.files;
                if (files.length > 0) {
                    fileInput.files = files;
                    fileInput.dispatchEvent(new Event('change'));
                }
            });
        }
        
        async function handlePdfFile(event) {
            const file = event.target.files[0];
            if (!file) return;
            
            const pdfInfo = document.getElementById('pdfInfo');
            pdfInfo.style.display = 'block';
            pdfInfo.innerHTML = `📄 ${file.name} (${(file.size / 1024 / 1024).toFixed(2)} MB)`;
            
            try {
                const arrayBuffer = await file.arrayBuffer();
                const pdf = await pdfjsLib.getDocument(arrayBuffer).promise;
                
                let fullText = '';
                for (let i = 1; i <= pdf.numPages; i++) {
                    const page = await pdf.getPage(i);
                    const textContent = await page.getTextContent();
                    const pageText = textContent.items.map(item => item.str).join(' ');
                    fullText += pageText + ' ';
                }
                
                pdfData = fullText;
                checkReadyToCompare();
            } catch (error) {
                pdfInfo.innerHTML = '❌ Error al leer el PDF';
                console.error('Error:', error);
            }
        }
        
        function handleExcelFile(event) {
            const file = event.target.files[0];
            if (!file) return;
            
            const excelInfo = document.getElementById('excelInfo');
            excelInfo.style.display = 'block';
            excelInfo.innerHTML = `📊 ${file.name} (${(file.size / 1024 / 1024).toFixed(2)} MB)`;
            
            const reader = new FileReader();
            reader.onload = function(e) {
                try {
                    const data = new Uint8Array(e.target.result);
                    const workbook = XLSX.read(data, { type: 'array' });
                    const firstSheet = workbook.Sheets[workbook.SheetNames[0]];
                    const jsonData = XLSX.utils.sheet_to_json(firstSheet);
                    
                    excelData = jsonData;
                    excelInfo.innerHTML += ` - ${jsonData.length} filas encontradas`;
                    checkReadyToCompare();
                } catch (error) {
                    excelInfo.innerHTML = '❌ Error al leer el Excel';
                    console.error('Error:', error);
                }
            };
            reader.readAsArrayBuffer(file);
        }
        
        function checkReadyToCompare() {
            const compareBtn = document.getElementById('compareBtn');
            if (pdfData && excelData) {
                compareBtn.disabled = false;
            }
        }
        
        async function compareFiles() {
            const resultsSection = document.getElementById('resultsSection');
            const loadingDiv = document.getElementById('loadingDiv');
            const resultsContent = document.getElementById('resultsContent');
            
            resultsSection.style.display = 'block';
            loadingDiv.style.display = 'block';
            resultsContent.style.display = 'none';
            
            // Simular tiempo de procesamiento
            await new Promise(resolve => setTimeout(resolve, 2000));
            
            const results = analyzeDiscrepancies();
            displayResults(results);
            
            loadingDiv.style.display = 'none';
            resultsContent.style.display = 'block';
        }
        
        function analyzeDiscrepancies() {
            const discrepancies = [];
            const matches = [];
            const missing = [];
            
            // Normalizar texto del PDF para búsqueda
            const pdfTextLower = pdfData.toLowerCase();
            
            excelData.forEach(row => {
                const sku = row.SKU || row.sku || row.Sku || '';
                const nombre = row.Nombre || row.nombre || row.NOMBRE || row.Name || row.name || '';
                const precio = row.Precio || row.precio || row.PRECIO || row.Price || row.price || '';
                const descripcion = row.Descripcion || row.descripcion || row.DESCRIPCION || row.Description || row.description || '';
                
                if (!sku) return;
                
                // Buscar SKU en el PDF
                const skuFound = pdfTextLower.includes(sku.toString().toLowerCase());
                
                if (!skuFound) {
                    missing.push({ sku, nombre, precio, descripcion });
                    return;
                }
                
                // Verificar discrepancias en nombre, precio y descripción
                const issues = [];
                
                if (nombre && !pdfTextLower.includes(nombre.toLowerCase())) {
                    issues.push({
                        field: 'Nombre',
                        expected: nombre,
                        status: 'No encontrado en PDF'
                    });
                }
                
                if (precio && !pdfTextLower.includes(precio.toString())) {
                    issues.push({
                        field: 'Precio',
                        expected: precio,
                        status: 'No encontrado en PDF'
                    });
                }
                
                if (descripcion && !pdfTextLower.includes(descripcion.toLowerCase())) {
                    issues.push({
                        field: 'Descripción',
                        expected: descripcion,
                        status: 'No encontrada en PDF'
                    });
                }
                
                if (issues.length > 0) {
                    discrepancies.push({
                        sku,
                        nombre,
                        precio,
                        descripcion,
                        issues
                    });
                } else {
                    matches.push({ sku, nombre, precio, descripcion });
                }
            });
            
            return { discrepancies, matches, missing };
        }
        
        function displayResults(results) {
            document.getElementById('matchesCount').textContent = results.matches.length;
            document.getElementById('discrepanciesCount').textContent = results.discrepancies.length;
            document.getElementById('missingCount').textContent = results.missing.length;
            
            const discrepanciesList = document.getElementById('discrepanciesList');
            
            if (results.discrepancies.length === 0 && results.missing.length === 0) {
                discrepanciesList.innerHTML = `
                    <div style="text-align: center; padding: 40px; color: #10b981;">
                        <div style="font-size: 3rem; margin-bottom: 20px;">✅</div>
                        <h3>¡Perfecto! No se encontraron discrepancias</h3>
                        <p>Todos los datos del Excel coinciden con el PDF</p>
                    </div>
                `;
                return;
            }
            
            let html = '';
            
            // Mostrar elementos faltantes
            results.missing.forEach(item => {
                html += `
                    <div class="discrepancy-item" style="border-left-color: #f59e0b; background: #fffbeb;">
                        <div class="discrepancy-sku" style="color: #d97706;">
                            SKU: ${item.sku} - ❌ NO ENCONTRADO EN PDF
                        </div>
                        <div class="discrepancy-details">
                            <div class="detail-column">
                                <h4>Datos del Excel</h4>
                                <div class="detail-item">Nombre: ${item.nombre || 'N/A'}</div>
                                <div class="detail-item">Precio: ${item.precio || 'N/A'}</div>
                                <div class="detail-item">Descripción: ${item.descripcion || 'N/A'}</div>
                            </div>
                            <div class="detail-column">
                                <h4>Estado</h4>
                                <div class="detail-item" style="color: #d97706;">SKU no encontrado en el PDF</div>
                            </div>
                        </div>
                    </div>
                `;
            });
            
            // Mostrar discrepancias
            results.discrepancies.forEach(item => {
                html += `
                    <div class="discrepancy-item">
                        <div class="discrepancy-sku">SKU: ${item.sku}</div>
                        <div class="discrepancy-details">
                            <div class="detail-column">
                                <h4>Datos del Excel</h4>
                                <div class="detail-item">Nombre: ${item.nombre || 'N/A'}</div>
                                <div class="detail-item">Precio: ${item.precio || 'N/A'}</div>
                                <div class="detail-item">Descripción: ${item.descripcion || 'N/A'}</div>
                            </div>
                            <div class="detail-column">
                                <h4>Problemas Encontrados</h4>
                                ${item.issues.map(issue => `
                                    <div class="detail-item" style="color: #dc2626;">
                                        ${issue.field}: ${issue.status}
                                    </div>
                                `).join('')}
                            </div>
                        </div>
                    </div>
                `;
            });
            
            discrepanciesList.innerHTML = html;
        }
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'969fbaf850ce8c3d',t:'MTc1NDMyODA3OC4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
