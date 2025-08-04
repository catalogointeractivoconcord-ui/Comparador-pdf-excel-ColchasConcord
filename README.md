<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>📊 Comparador PDF vs Excel</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .file-drop-zone {
            border: 2px dashed #cbd5e0;
            transition: all 0.3s ease;
        }
        .file-drop-zone:hover {
            border-color: #4299e1;
            background-color: #ebf8ff;
        }
        .file-drop-zone.dragover {
            border-color: #3182ce;
            background-color: #bee3f8;
        }
        .pulse-animation {
            animation: pulse 2s infinite;
        }
        .progress-bar {
            transition: width 0.3s ease;
        }
    </style>
</head>
<body class="bg-gradient-to-br from-blue-50 to-indigo-100 min-h-screen">
    <div class="container mx-auto px-4 py-8">
        <!-- Header -->
        <div class="text-center mb-8">
            <h1 class="text-4xl font-bold text-gray-800 mb-2">📊 Comparador PDF vs Excel</h1>
            <p class="text-gray-600 text-lg">Detecta discrepancias entre tu catálogo PDF y base de datos Excel</p>
        </div>

        <!-- File Upload Section -->
        <div class="grid md:grid-cols-2 gap-8 mb-8">
            <!-- PDF Upload -->
            <div class="bg-white rounded-lg shadow-lg p-6">
                <h2 class="text-xl font-semibold mb-4 text-gray-700">📄 Subir archivo PDF</h2>
                <div id="pdfDropZone" class="file-drop-zone rounded-lg p-8 text-center cursor-pointer">
                    <div class="mb-4">
                        <svg class="mx-auto h-12 w-12 text-gray-400" stroke="currentColor" fill="none" viewBox="0 0 48 48">
                            <path d="M28 8H12a4 4 0 00-4 4v20m32-12v8m0 0v8a4 4 0 01-4 4H12a4 4 0 01-4-4v-4m32-4l-3.172-3.172a4 4 0 00-5.656 0L28 28M8 32l9.172-9.172a4 4 0 015.656 0L28 28m0 0l4 4m4-24h8m-4-4v8m-12 4h.02" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                        </svg>
                    </div>
                    <p class="text-gray-600 mb-2">Arrastra tu PDF aquí o haz clic para seleccionar</p>
                    <p class="text-sm text-gray-400">Formatos soportados: PDF</p>
                </div>
                <input type="file" id="pdfInput" accept=".pdf" class="hidden">
                <div id="pdfStatus" class="mt-4 text-sm"></div>
            </div>

            <!-- Excel Upload -->
            <div class="bg-white rounded-lg shadow-lg p-6">
                <h2 class="text-xl font-semibold mb-4 text-gray-700">📊 Subir archivo Excel</h2>
                <div id="excelDropZone" class="file-drop-zone rounded-lg p-8 text-center cursor-pointer">
                    <div class="mb-4">
                        <svg class="mx-auto h-12 w-12 text-gray-400" stroke="currentColor" fill="none" viewBox="0 0 48 48">
                            <path d="M9 12h6l3 9 3-9h6m-6 0v12m-6-6h12M21 21v-9m0 9l3 9m-3-9h6m0 0v9" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                        </svg>
                    </div>
                    <p class="text-gray-600 mb-2">Arrastra tu Excel aquí o haz clic para seleccionar</p>
                    <p class="text-sm text-gray-400">Formatos soportados: XLSX, XLS</p>
                    <p class="text-xs text-blue-600 mt-2">Columnas requeridas: SKU, Nombre, Precio, Descripcion</p>
                </div>
                <input type="file" id="excelInput" accept=".xlsx,.xls" class="hidden">
                <div id="excelStatus" class="mt-4 text-sm"></div>
            </div>
        </div>

        <!-- Configuration Panel -->
        <div class="bg-white rounded-lg shadow-lg p-6 mb-8">
            <h2 class="text-xl font-semibold mb-4 text-gray-700">⚙️ Configuración de Análisis</h2>
            <div class="grid md:grid-cols-3 gap-4">
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-2">Tolerancia de Precio (%):</label>
                    <input type="number" id="priceTolerance" value="5" min="0" max="50" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500">
                    <p class="text-xs text-gray-500 mt-1">Diferencia permitida en precios</p>
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-2">Umbral de Nombre (%):</label>
                    <input type="number" id="nameThreshold" value="40" min="0" max="100" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500">
                    <p class="text-xs text-gray-500 mt-1">Similitud mínima para nombres</p>
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-2">Umbral Total (%):</label>
                    <input type="number" id="totalThreshold" value="60" min="0" max="100" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500">
                    <p class="text-xs text-gray-500 mt-1">Puntuación mínima para aprobar</p>
                </div>
            </div>
        </div>

        <!-- Compare Button -->
        <div class="text-center mb-8">
            <button id="compareBtn" class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-4 px-8 rounded-lg text-lg shadow-lg transform transition hover:scale-105 disabled:opacity-50 disabled:cursor-not-allowed">
                🔍 Comparar Archivos
            </button>
            <div id="progressContainer" class="hidden mt-4">
                <div class="bg-gray-200 rounded-full h-2">
                    <div id="progressBar" class="bg-blue-600 h-2 rounded-full progress-bar" style="width: 0%"></div>
                </div>
                <p id="progressText" class="text-sm text-gray-600 mt-2">Iniciando análisis...</p>
            </div>
        </div>

        <!-- Results Section -->
        <div id="results"></div>
    </div>

    <script>
        let pdfData = null;
        let excelData = null;

        // PDF.js worker setup
        pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.worker.min.js';

        // File upload handlers
        function setupFileHandlers() {
            // PDF handlers
            const pdfDropZone = document.getElementById('pdfDropZone');
            const pdfInput = document.getElementById('pdfInput');
            
            pdfDropZone.addEventListener('click', () => pdfInput.click());
            pdfDropZone.addEventListener('dragover', handleDragOver);
            pdfDropZone.addEventListener('drop', (e) => handleFileDrop(e, 'pdf'));
            pdfInput.addEventListener('change', (e) => handleFileSelect(e, 'pdf'));

            // Excel handlers
            const excelDropZone = document.getElementById('excelDropZone');
            const excelInput = document.getElementById('excelInput');
            
            excelDropZone.addEventListener('click', () => excelInput.click());
            excelDropZone.addEventListener('dragover', handleDragOver);
            excelDropZone.addEventListener('drop', (e) => handleFileDrop(e, 'excel'));
            excelInput.addEventListener('change', (e) => handleFileSelect(e, 'excel'));

            // Compare button
            document.getElementById('compareBtn').addEventListener('click', comparePDFWithExcel);
        }

        function handleDragOver(e) {
            e.preventDefault();
            e.currentTarget.classList.add('dragover');
        }

        function handleFileDrop(e, type) {
            e.preventDefault();
            e.currentTarget.classList.remove('dragover');
            const files = e.dataTransfer.files;
            if (files.length > 0) {
                processFile(files[0], type);
            }
        }

        function handleFileSelect(e, type) {
            const files = e.target.files;
            if (files.length > 0) {
                processFile(files[0], type);
            }
        }

        function processFile(file, type) {
            const reader = new FileReader();
            
            reader.onload = function(e) {
                if (type === 'pdf') {
                    pdfData = new Uint8Array(e.target.result);
                    document.getElementById('pdfStatus').innerHTML = 
                        '<span class="text-green-600">✅ PDF cargado: ' + file.name + ' (' + (file.size / 1024 / 1024).toFixed(2) + ' MB)</span>';
                } else {
                    excelData = new Uint8Array(e.target.result);
                    document.getElementById('excelStatus').innerHTML = 
                        '<span class="text-green-600">✅ Excel cargado: ' + file.name + ' (' + (file.size / 1024).toFixed(2) + ' KB)</span>';
                }
                
                // Enable compare button if both files are loaded
                if (pdfData && excelData) {
                    document.getElementById('compareBtn').classList.remove('opacity-50');
                    document.getElementById('compareBtn').classList.add('pulse-animation');
                }
            };
            
            reader.readAsArrayBuffer(file);
        }

        function updateProgress(percent, text) {
            const progressContainer = document.getElementById('progressContainer');
            const progressBar = document.getElementById('progressBar');
            const progressText = document.getElementById('progressText');
            
            progressContainer.classList.remove('hidden');
            progressBar.style.width = percent + '%';
            progressText.textContent = text;
        }

        async function extractTextFromPDF(pdfData) {
            updateProgress(10, 'Extrayendo texto del PDF...');
            
            const pdf = await pdfjsLib.getDocument(pdfData).promise;
            let fullText = '';
            
            for (let i = 1; i <= pdf.numPages; i++) {
                updateProgress(10 + (i / pdf.numPages) * 30, `Procesando página ${i} de ${pdf.numPages}...`);
                const page = await pdf.getPage(i);
                const textContent = await page.getTextContent();
                const pageText = textContent.items.map(item => item.str).join(' ');
                fullText += pageText + '\n';
            }
            
            return fullText;
        }

        function extractNumericPrice(priceStr) {
            if (!priceStr) return null;
            const cleaned = String(priceStr).replace(/[^\d.,]/g, '');
            const numeric = parseFloat(cleaned.replace(/,/g, ''));
            return isNaN(numeric) ? null : numeric;
        }

        function calculateTextSimilarity(text1, text2) {
            if (!text1 || !text2) return 0;
            
            const words1 = text1.toLowerCase().split(/\s+/).filter(w => w.length > 2);
            const words2 = text2.toLowerCase().split(/\s+/).filter(w => w.length > 2);
            
            const commonWords = ['de', 'la', 'el', 'en', 'con', 'para', 'por', 'un', 'una', 'del', 'las', 'los', 'y', 'o'];
            const filteredWords1 = words1.filter(w => !commonWords.includes(w));
            const filteredWords2 = words2.filter(w => !commonWords.includes(w));
            
            if (filteredWords1.length === 0) return 0;
            
            let matches = 0;
            filteredWords1.forEach(word1 => {
                if (filteredWords2.some(word2 => 
                    word2.includes(word1) || 
                    word1.includes(word2) || 
                    levenshteinDistance(word1, word2) <= 2
                )) {
                    matches++;
                }
            });
            
            return (matches / filteredWords1.length) * 100;
        }

        function levenshteinDistance(str1, str2) {
            const matrix = [];
            for (let i = 0; i <= str2.length; i++) {
                matrix[i] = [i];
            }
            for (let j = 0; j <= str1.length; j++) {
                matrix[0][j] = j;
            }
            for (let i = 1; i <= str2.length; i++) {
                for (let j = 1; j <= str1.length; j++) {
                    if (str2.charAt(i - 1) === str1.charAt(j - 1)) {
                        matrix[i][j] = matrix[i - 1][j - 1];
                    } else {
                        matrix[i][j] = Math.min(
                            matrix[i - 1][j - 1] + 1,
                            matrix[i][j - 1] + 1,
                            matrix[i - 1][j] + 1
                        );
                    }
                }
            }
            return matrix[str2.length][str1.length];
        }

        async function analyzeData(pdfText, excelRows) {
            const results = {
                matches: [],
                discrepancies: [],
                notFound: [],
                debug: [],
                pdfAnalysis: {},
                summary: {}
            };

            // CONFIGURACIÓN ULTRA AGRESIVA PARA DETECTAR TODAS LAS DISCREPANCIAS
            const PRICE_TOLERANCE = 0.01; // Solo 1% de tolerancia en precios
            const NAME_MATCH_THRESHOLD = 20; // Muy bajo - cualquier cosa menor es discrepancia
            const DESCRIPTION_MATCH_THRESHOLD = 15; // Muy bajo
            const OVERALL_SCORE_THRESHOLD = 40; // Muy bajo - casi todo será discrepancia

            updateProgress(50, 'Analizando contenido del PDF con máxima sensibilidad...');

            // Análisis exhaustivo del PDF
            const pdfLines = pdfText.split('\n').filter(line => line.trim().length > 0);
            const cleanPdfText = pdfText.toLowerCase();
            const pdfSkuPatterns = pdfText.match(/[A-Z0-9]{2,}[-_]?[A-Z0-9]{1,}/gi) || [];
            const pdfPrices = pdfText.match(/\$?\s*[\d,]+\.?\d*|\$?\s*[\d.]+,?\d*/g) || [];
            
            // Crear índice de palabras del PDF para búsqueda rápida
            const pdfWords = new Set();
            cleanPdfText.split(/\s+/).forEach(word => {
                if (word.length > 2) {
                    pdfWords.add(word.replace(/[^\w]/g, ''));
                }
            });
            
            results.pdfAnalysis = {
                totalLines: pdfLines.length,
                possibleSkus: pdfSkuPatterns.slice(0, 30),
                possiblePrices: pdfPrices.slice(0, 30),
                textLength: pdfText.length,
                uniqueWords: pdfWords.size,
                settings: {
                    priceTolerancePercent: (PRICE_TOLERANCE * 100).toFixed(1),
                    nameMatchThreshold: NAME_MATCH_THRESHOLD,
                    descriptionMatchThreshold: DESCRIPTION_MATCH_THRESHOLD,
                    overallScoreThreshold: OVERALL_SCORE_THRESHOLD
                }
            };

            results.debug.push(`🔥 CONFIGURACIÓN ULTRA AGRESIVA:`);
            results.debug.push(`• Tolerancia de precio: ${(PRICE_TOLERANCE * 100).toFixed(1)}% (EXTREMA)`);
            results.debug.push(`• Umbral de nombre: ${NAME_MATCH_THRESHOLD}% (MUY BAJO)`);
            results.debug.push(`• Umbral descripción: ${DESCRIPTION_MATCH_THRESHOLD}% (MUY BAJO)`);
            results.debug.push(`• Umbral total: ${OVERALL_SCORE_THRESHOLD}% (MUY BAJO)`);
            results.debug.push(`📄 PDF: ${pdfLines.length} líneas, ${pdfSkuPatterns.length} SKUs, ${pdfWords.size} palabras únicas`);

            // Procesar cada fila del Excel con análisis exhaustivo
            for (let index = 0; index < excelRows.length; index++) {
                const progress = 50 + ((index / excelRows.length) * 40);
                updateProgress(progress, `Análisis exhaustivo SKU ${index + 1} de ${excelRows.length}...`);

                const row = excelRows[index];
                const sku = String(row.SKU || row.sku || row.Sku || row.codigo || row.Codigo || '').trim();
                const nombre = String(row.Nombre || row.nombre || row.Name || row.name || row.Producto || row.producto || '').trim();
                const precio = String(row.Precio || row.precio || row.Price || row.price || '').trim();
                const descripcion = String(row.Descripcion || row.descripcion || row.Description || row.description || '').trim();

                if (!sku) {
                    results.debug.push(`⚠️ Fila ${index + 1}: No se encontró SKU válido`);
                    continue;
                }

                results.debug.push(`\n🔍 ANÁLISIS EXHAUSTIVO SKU: ${sku}`);

                // Generar MUCHAS más variaciones del SKU
                const skuVariations = [
                    sku,
                    sku.replace(/[-_\s]/g, ''),
                    sku.replace(/[-_\s]/g, '').toUpperCase(),
                    sku.replace(/[-_\s]/g, '').toLowerCase(),
                    sku.toUpperCase(),
                    sku.toLowerCase(),
                    sku.replace(/0/g, 'O'),
                    sku.replace(/O/g, '0'),
                    sku.replace(/1/g, 'I'),
                    sku.replace(/I/g, '1'),
                    sku.replace(/5/g, 'S'),
                    sku.replace(/S/g, '5'),
                    sku.replace(/[-_]/g, ''),
                    sku.replace(/[-_]/g, ' '),
                    sku.split('').join(' '), // Separado por espacios
                    sku.split('').join(''), // Sin separadores
                    sku.substring(0, sku.length - 1), // Sin último carácter
                    sku.substring(1), // Sin primer carácter
                ];

                let skuFound = false;
                let foundContext = '';
                let bestMatch = '';
                let allFoundContexts = [];

                // Búsqueda ULTRA exhaustiva del SKU
                for (let variation of skuVariations) {
                    if (variation.length < 2) continue;
                    
                    // Búsqueda exacta
                    const escapedVariation = variation.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
                    const exactRegex = new RegExp(`\\b${escapedVariation}\\b`, 'gi');
                    
                    if (exactRegex.test(pdfText)) {
                        skuFound = true;
                        bestMatch = variation;
                        
                        // Extraer contexto MUY amplio (1200 caracteres)
                        const contextRegex = new RegExp(`(.{0,1200}\\b${escapedVariation}\\b.{0,1200})`, 'gi');
                        const matches = pdfText.match(contextRegex);
                        if (matches) {
                            foundContext = matches[0];
                            allFoundContexts.push(matches[0]);
                        }
                        
                        results.debug.push(`✅ SKU encontrado EXACTO: "${variation}"`);
                        break;
                    }
                    
                    // Búsqueda parcial (más flexible)
                    if (!skuFound && variation.length >= 4) {
                        const partialRegex = new RegExp(escapedVariation, 'gi');
                        if (partialRegex.test(pdfText)) {
                            skuFound = true;
                            bestMatch = variation + ' (parcial)';
                            
                            const contextRegex = new RegExp(`(.{0,1200}${escapedVariation}.{0,1200})`, 'gi');
                            const matches = pdfText.match(contextRegex);
                            if (matches) {
                                foundContext = matches[0];
                                allFoundContexts.push(matches[0]);
                            }
                            
                            results.debug.push(`⚠️ SKU encontrado PARCIAL: "${variation}"`);
                            break;
                        }
                    }
                }

                if (skuFound) {
                    const issues = [];
                    const scores = { nombre: 0, precio: 0, descripcion: 0 };
                    const detailedAnalysis = [];
                    
                    // Combinar todos los contextos encontrados
                    const fullContext = allFoundContexts.join(' ');
                    const contextToAnalyze = fullContext || foundContext;
                    
                    results.debug.push(`📝 Contexto total (${contextToAnalyze.length} chars): "${contextToAnalyze.substring(0, 300)}..."`);

                    // ANÁLISIS DE NOMBRE ULTRA DETALLADO
                    if (nombre && nombre.length > 1) {
                        const similarity = calculateTextSimilarity(nombre, contextToAnalyze);
                        scores.nombre = similarity;
                        
                        // Análisis palabra por palabra
                        const nombreWords = nombre.toLowerCase().split(/\s+/).filter(w => w.length > 2);
                        const contextLower = contextToAnalyze.toLowerCase();
                        
                        let wordsFound = 0;
                        let wordsNotFound = [];
                        
                        nombreWords.forEach(word => {
                            if (contextLower.includes(word)) {
                                wordsFound++;
                            } else {
                                wordsNotFound.push(word);
                            }
                        });
                        
                        const wordMatchPercent = nombreWords.length > 0 ? (wordsFound / nombreWords.length) * 100 : 0;
                        
                        results.debug.push(`📛 Análisis nombre detallado: "${nombre}"`);
                        results.debug.push(`   • Similitud general: ${similarity.toFixed(1)}%`);
                        results.debug.push(`   • Palabras encontradas: ${wordsFound}/${nombreWords.length} (${wordMatchPercent.toFixed(1)}%)`);
                        results.debug.push(`   • Palabras NO encontradas: ${wordsNotFound.join(', ')}`);
                        
                        if (similarity < NAME_MATCH_THRESHOLD) {
                            issues.push(`❌ NOMBRE: "${nombre}" coincidencia muy baja (${similarity.toFixed(1)}%)`);
                            issues.push(`   • Solo ${wordsFound}/${nombreWords.length} palabras encontradas`);
                            if (wordsNotFound.length > 0) {
                                issues.push(`   • Palabras faltantes: ${wordsNotFound.join(', ')}`);
                            }
                        }
                        
                        // Penalizar más si faltan palabras clave
                        if (wordMatchPercent < 50) {
                            issues.push(`❌ CRÍTICO: Menos del 50% de las palabras del nombre encontradas`);
                        }
                    } else {
                        scores.nombre = 100;
                    }

                    // ANÁLISIS DE PRECIO ULTRA ESTRICTO
                    if (precio && precio.length > 0) {
                        const expectedPrice = extractNumericPrice(precio);
                        const contextPrices = contextToAnalyze.match(/\$?\s*[\d,]+\.?\d*|\$?\s*[\d.]+,?\d*/g) || [];
                        
                        let priceMatch = false;
                        let closestPrice = null;
                        let priceDifference = Infinity;
                        let allPricesFound = [];
                        
                        if (expectedPrice) {
                            contextPrices.forEach(contextPriceStr => {
                                const contextPrice = extractNumericPrice(contextPriceStr);
                                if (contextPrice) {
                                    allPricesFound.push(contextPrice);
                                    const difference = Math.abs(expectedPrice - contextPrice) / expectedPrice;
                                    if (difference < priceDifference) {
                                        priceDifference = difference;
                                        closestPrice = contextPrice;
                                    }
                                    if (difference <= PRICE_TOLERANCE) {
                                        priceMatch = true;
                                    }
                                }
                            });
                        }
                        
                        results.debug.push(`💰 Análisis precio detallado: "${precio}" (${expectedPrice})`);
                        results.debug.push(`   • Precios en contexto: ${allPricesFound.join(', ')}`);
                        results.debug.push(`   • Precio más cercano: ${closestPrice} (diferencia: ${(priceDifference * 100).toFixed(2)}%)`);
                        
                        if (priceMatch) {
                            scores.precio = 100;
                            results.debug.push(`   ✅ PRECIO COINCIDE (tolerancia: ${(PRICE_TOLERANCE * 100).toFixed(1)}%)`);
                        } else {
                            scores.precio = 0;
                            issues.push(`❌ PRECIO: Esperado "${precio}" (${expectedPrice})`);
                            if (closestPrice) {
                                const diffPercent = (priceDifference * 100).toFixed(2);
                                issues.push(`   • Más cercano: ${closestPrice} (diferencia: ${diffPercent}%)`);
                                if (priceDifference > 0.05) { // Más del 5%
                                    issues.push(`   🚨 DIFERENCIA SIGNIFICATIVA: Más del 5%`);
                                }
                            } else {
                                issues.push(`   🚨 CRÍTICO: No se encontró ningún precio en el contexto`);
                            }
                            issues.push(`   • Todos los precios encontrados: ${allPricesFound.join(', ')}`);
                        }
                    } else {
                        scores.precio = 100;
                    }

                    // ANÁLISIS DE DESCRIPCIÓN ULTRA DETALLADO
                    if (descripcion && descripcion.length > 3) {
                        const similarity = calculateTextSimilarity(descripcion, contextToAnalyze);
                        scores.descripcion = similarity;
                        
                        // Análisis de características específicas
                        const descWords = descripcion.toLowerCase().split(/\s+/).filter(w => w.length > 2);
                        const contextLower = contextToAnalyze.toLowerCase();
                        
                        let descWordsFound = 0;
                        let descWordsNotFound = [];
                        
                        descWords.forEach(word => {
                            if (contextLower.includes(word)) {
                                descWordsFound++;
                            } else {
                                descWordsNotFound.push(word);
                            }
                        });
                        
                        const descWordMatchPercent = descWords.length > 0 ? (descWordsFound / descWords.length) * 100 : 0;
                        
                        results.debug.push(`📝 Análisis descripción detallado: "${descripcion}"`);
                        results.debug.push(`   • Similitud general: ${similarity.toFixed(1)}%`);
                        results.debug.push(`   • Palabras encontradas: ${descWordsFound}/${descWords.length} (${descWordMatchPercent.toFixed(1)}%)`);
                        
                        if (similarity < DESCRIPTION_MATCH_THRESHOLD) {
                            issues.push(`⚠️ DESCRIPCIÓN: "${descripcion}" coincidencia baja (${similarity.toFixed(1)}%)`);
                            if (descWordsNotFound.length > 0) {
                                issues.push(`   • Palabras faltantes: ${descWordsNotFound.join(', ')}`);
                            }
                        }
                    } else {
                        scores.descripcion = 100;
                    }

                    // CÁLCULO DE PUNTUACIÓN TOTAL CON PESOS
                    // Dar más peso al precio y nombre
                    const weightedScore = (scores.nombre * 0.4) + (scores.precio * 0.5) + (scores.descripcion * 0.1);
                    const totalScore = (scores.nombre + scores.precio + scores.descripcion) / 3;
                    
                    results.debug.push(`📊 PUNTUACIÓN DETALLADA:`);
                    results.debug.push(`   • Nombre: ${scores.nombre.toFixed(1)}% (peso 40%)`);
                    results.debug.push(`   • Precio: ${scores.precio.toFixed(1)}% (peso 50%)`);
                    results.debug.push(`   • Descripción: ${scores.descripcion.toFixed(1)}% (peso 10%)`);
                    results.debug.push(`   • Promedio simple: ${totalScore.toFixed(1)}%`);
                    results.debug.push(`   • Promedio ponderado: ${weightedScore.toFixed(1)}%`);

                    // DECISIÓN FINAL MUY ESTRICTA
                    const hasDiscrepancy = weightedScore < OVERALL_SCORE_THRESHOLD || 
                                         issues.length > 0 || 
                                         scores.precio < 100 || 
                                         scores.nombre < NAME_MATCH_THRESHOLD;
                    
                    if (hasDiscrepancy) {
                        // Determinar severidad más precisa
                        let severity = 'low';
                        if (scores.precio < 100 || weightedScore < 20) severity = 'high';
                        else if (scores.nombre < 30 || weightedScore < 40) severity = 'medium';
                        
                        results.discrepancies.push({
                            sku,
                            matchedAs: bestMatch,
                            expected: { nombre, precio, descripcion },
                            found: contextToAnalyze,
                            issues,
                            scores,
                            totalScore: totalScore.toFixed(1),
                            weightedScore: weightedScore.toFixed(1),
                            severity,
                            analysis: detailedAnalysis
                        });
                        results.debug.push(`🚨 DISCREPANCIA DETECTADA: ${sku} (ponderado: ${weightedScore.toFixed(1)}%)`);
                    } else {
                        results.matches.push({
                            sku,
                            matchedAs: bestMatch,
                            data: { nombre, precio, descripcion },
                            context: contextToAnalyze.substring(0, 300),
                            scores,
                            totalScore: totalScore.toFixed(1),
                            weightedScore: weightedScore.toFixed(1)
                        });
                        results.debug.push(`✅ COINCIDENCIA PERFECTA: ${sku} (ponderado: ${weightedScore.toFixed(1)}%)`);
                    }
                } else {
                    results.debug.push(`❌ SKU ${sku} COMPLETAMENTE NO ENCONTRADO`);
                    results.debug.push(`   • Variaciones probadas: ${skuVariations.slice(0, 8).join(', ')}`);
                    results.notFound.push({
                        sku,
                        data: { nombre, precio, descripcion },
                        searchedVariations: skuVariations.slice(0, 10)
                    });
                }
            }

            // Generar resumen con métricas adicionales
            const totalProcessed = results.matches.length + results.discrepancies.length + results.notFound.length;
            results.summary = {
                total: excelRows.length,
                totalProcessed,
                matches: results.matches.length,
                discrepancies: results.discrepancies.length,
                notFound: results.notFound.length,
                successRate: totalProcessed > 0 ? ((results.matches.length / totalProcessed) * 100).toFixed(1) : '0',
                discrepancyRate: totalProcessed > 0 ? ((results.discrepancies.length / totalProcessed) * 100).toFixed(1) : '0',
                notFoundRate: totalProcessed > 0 ? ((results.notFound.length / totalProcessed) * 100).toFixed(1) : '0'
            };

            updateProgress(100, 'Análisis ultra exhaustivo completado!');
            
            return results;
        }

        async function comparePDFWithExcel() {
            if (!pdfData || !excelData) {
                alert('❌ Por favor sube ambos archivos primero');
                return;
            }

            const compareBtn = document.getElementById('compareBtn');
            const originalText = compareBtn.textContent;
            compareBtn.textContent = '🔄 Analizando...';
            compareBtn.disabled = true;
            compareBtn.classList.remove('pulse-animation');

            try {
                // Extract PDF text
                const pdfText = await extractTextFromPDF(pdfData);
                console.log('✅ PDF text extracted:', pdfText.length, 'characters');

                // Process Excel
                updateProgress(40, 'Procesando archivo Excel...');
                const workbook = XLSX.read(excelData, { type: 'array' });
                const worksheet = workbook.Sheets[workbook.SheetNames[0]];
                const excelRows = XLSX.utils.sheet_to_json(worksheet);
                console.log('✅ Excel data processed:', excelRows.length, 'rows');

                if (excelRows.length === 0) {
                    throw new Error('El archivo Excel está vacío o no tiene el formato correcto');
                }

                // Analyze data
                const results = await analyzeData(pdfText, excelRows);
                console.log('✅ Analysis completed:', results);

                // Display results
                displayResults(results);

            } catch (error) {
                console.error('❌ Error during comparison:', error);
                alert('❌ Error al procesar los archivos: ' + error.message);
                document.getElementById('results').innerHTML = `
                    <div class="bg-red-50 border border-red-200 rounded-lg p-6">
                        <h3 class="text-lg font-semibold text-red-800 mb-2">❌ Error en el Análisis</h3>
                        <p class="text-red-600">${error.message}</p>
                        <p class="text-sm text-red-500 mt-2">Verifica que los archivos estén en el formato correcto y vuelve a intentar.</p>
                    </div>
                `;
            } finally {
                compareBtn.textContent = originalText;
                compareBtn.disabled = false;
                document.getElementById('progressContainer').classList.add('hidden');
            }
        }

        function displayResults(results) {
            const resultsDiv = document.getElementById('results');
            
            let html = '<div class="bg-white rounded-lg shadow-lg p-6 mt-6">';
            html += '<h2 class="text-2xl font-bold mb-4 text-gray-800">📊 Resultados del Análisis</h2>';

            // Summary with enhanced metrics
            html += '<div class="bg-gradient-to-r from-blue-50 to-indigo-50 border border-blue-200 rounded-lg p-6 mb-6">';
            html += '<h3 class="text-lg font-semibold text-blue-800 mb-4">📋 Resumen Ejecutivo</h3>';
            html += '<div class="grid grid-cols-2 md:grid-cols-5 gap-4 mb-4">';
            html += `<div class="text-center p-3 bg-white rounded-lg shadow-sm">
                        <div class="text-2xl font-bold text-blue-600">${results.summary.total}</div>
                        <div class="text-sm text-gray-600">Total SKUs</div>
                     </div>`;
            html += `<div class="text-center p-3 bg-white rounded-lg shadow-sm">
                        <div class="text-2xl font-bold text-green-600">${results.summary.matches}</div>
                        <div class="text-sm text-gray-600">✅ Correctos</div>
                     </div>`;
            html += `<div class="text-center p-3 bg-white rounded-lg shadow-sm">
                        <div class="text-2xl font-bold text-yellow-600">${results.summary.discrepancies}</div>
                        <div class="text-sm text-gray-600">⚠️ Discrepancias</div>
                     </div>`;
            html += `<div class="text-center p-3 bg-white rounded-lg shadow-sm">
                        <div class="text-2xl font-bold text-red-600">${results.summary.notFound}</div>
                        <div class="text-sm text-gray-600">❌ No encontrados</div>
                     </div>`;
            html += `<div class="text-center p-3 bg-white rounded-lg shadow-sm">
                        <div class="text-2xl font-bold text-purple-600">${results.summary.successRate}%</div>
                        <div class="text-sm text-gray-600">📈 Tasa de éxito</div>
                     </div>`;
            html += '</div>';
            
            // Progress bar for success rate
            html += '<div class="mb-4">';
            html += '<div class="flex justify-between text-sm text-gray-600 mb-1">';
            html += '<span>Tasa de Coincidencias</span>';
            html += `<span>${results.summary.successRate}%</span>`;
            html += '</div>';
            html += '<div class="bg-gray-200 rounded-full h-3">';
            html += `<div class="bg-gradient-to-r from-green-400 to-blue-500 h-3 rounded-full" style="width: ${results.summary.successRate}%"></div>`;
            html += '</div>';
            html += '</div>';
            html += '</div>';

            // PDF Analysis
            if (results.pdfAnalysis) {
                html += '<div class="bg-purple-50 border border-purple-200 rounded-lg p-4 mb-6">';
                html += '<h3 class="text-lg font-semibold text-purple-800 mb-3">🔍 Análisis del PDF</h3>';
                html += '<div class="grid md:grid-cols-3 gap-4 text-sm">';
                html += '<div class="bg-white p-3 rounded">';
                html += `<p><strong>📄 Líneas de texto:</strong> ${results.pdfAnalysis.totalLines.toLocaleString()}</p>`;
                html += `<p><strong>📝 Caracteres totales:</strong> ${results.pdfAnalysis.textLength.toLocaleString()}</p>`;
                html += `<p><strong>🏷️ SKUs detectados:</strong> ${results.pdfAnalysis.possibleSkus.length}</p>`;
                html += `<p><strong>💰 Precios detectados:</strong> ${results.pdfAnalysis.possiblePrices.length}</p>`;
                html += '</div>';
                html += '<div class="bg-white p-3 rounded">';
                html += '<p><strong>⚙️ Configuración:</strong></p>';
                html += `<p>• Tolerancia precio: ${results.pdfAnalysis.settings.priceTolerancePercent}%</p>`;
                html += `<p>• Umbral nombre: ${results.pdfAnalysis.settings.nameMatchThreshold}%</p>`;
                html += `<p>• Umbral total: ${results.pdfAnalysis.settings.overallScoreThreshold}%</p>`;
                html += '</div>';
                html += '<div class="bg-white p-3 rounded">';
                html += '<p><strong>🔍 Primeros SKUs encontrados:</strong></p>';
                html += '<div class="text-xs font-mono bg-gray-50 p-2 rounded mt-1 max-h-20 overflow-y-auto">';
                html += results.pdfAnalysis.possibleSkus.slice(0, 10).join(', ');
                html += '</div>';
                html += '</div>';
                html += '</div>';
                html += '</div>';
            }

            // Discrepancies with severity levels
            if (results.discrepancies.length > 0) {
                html += '<div class="mb-6">';
                html += `<h3 class="text-lg font-semibold text-yellow-600 mb-4">⚠️ Discrepancias Encontradas (${results.discrepancies.length})</h3>`;
                
                const highSeverity = results.discrepancies.filter(d => d.severity === 'high');
                const mediumSeverity = results.discrepancies.filter(d => d.severity === 'medium');
                const lowSeverity = results.discrepancies.filter(d => d.severity === 'low');

                if (highSeverity.length > 0) {
                    html += `<h4 class="text-red-600 font-semibold mb-3">🚨 Severidad Alta (${highSeverity.length})</h4>`;
                    highSeverity.forEach(disc => html += renderDiscrepancy(disc, 'red'));
                }

                if (mediumSeverity.length > 0) {
                    html += `<h4 class="text-yellow-600 font-semibold mb-3 mt-6">⚠️ Severidad Media (${mediumSeverity.length})</h4>`;
                    mediumSeverity.forEach(disc => html += renderDiscrepancy(disc, 'yellow'));
                }

                if (lowSeverity.length > 0) {
                    html += `<h4 class="text-orange-600 font-semibold mb-3 mt-6">⚡ Severidad Baja (${lowSeverity.length})</h4>`;
                    lowSeverity.forEach(disc => html += renderDiscrepancy(disc, 'orange'));
                }

                html += '</div>';
            }

            // Matches
            if (results.matches.length > 0) {
                html += '<div class="mb-6">';
                html += `<h3 class="text-lg font-semibold text-green-600 mb-3">✅ Coincidencias Correctas (${results.matches.length})</h3>`;
                html += '<div class="grid md:grid-cols-2 gap-3">';
                results.matches.forEach(match => {
                    html += '<div class="bg-green-50 border-l-4 border-green-400 rounded-lg p-4">';
                    html += '<div class="flex justify-between items-start mb-2">';
                    html += `<strong class="text-green-800">SKU: ${match.sku}</strong>`;
                    html += `<span class="text-xs bg-green-200 px-2 py-1 rounded">Puntuación: ${match.totalScore}%</span>`;
                    html += '</div>';
                    if (match.matchedAs !== match.sku) {
                        html += `<p class="text-xs text-green-600 mb-2">Encontrado como: <code>${match.matchedAs}</code></p>`;
                    }
                    html += '<div class="text-sm text-gray-600">';
                    html += `<p><strong>Nombre:</strong> ${match.data.nombre || 'N/A'}</p>`;
                    html += `<p><strong>Precio:</strong> ${match.data.precio || 'N/A'}</p>`;
                    html += '</div>';
                    html += '</div>';
                });
                html += '</div>';
                html += '</div>';
            }

            // Not found
            if (results.notFound.length > 0) {
                html += '<div class="mb-6">';
                html += `<h3 class="text-lg font-semibold text-red-600 mb-3">❌ SKUs No Encontrados (${results.notFound.length})</h3>`;
                html += '<div class="grid md:grid-cols-2 gap-3">';
                results.notFound.forEach(item => {
                    html += '<div class="bg-red-50 border-l-4 border-red-400 rounded-lg p-4">';
                    html += `<strong class="text-red-800">SKU: ${item.sku}</strong>`;
                    html += '<div class="text-sm text-gray-600 mt-2">';
                    html += `<p><strong>Nombre:</strong> ${item.data.nombre || 'N/A'}</p>`;
                    html += `<p><strong>Precio:</strong> ${item.data.precio || 'N/A'}</p>`;
                    if (item.searchedVariations) {
                        html += `<p class="text-xs text-gray-500 mt-2"><strong>Variaciones buscadas:</strong> ${item.searchedVariations.join(', ')}</p>`;
                    }
                    html += '</div>';
                    html += '</div>';
                });
                html += '</div>';
                html += '</div>';
            }

            // Debug information (collapsible)
            if (results.debug && results.debug.length > 0) {
                html += '<div class="bg-gray-50 border border-gray-200 rounded-lg p-4">';
                html += '<h3 class="text-lg font-semibold text-gray-700 mb-2 cursor-pointer" onclick="toggleDebug()">🔍 Información de Depuración (Click para expandir)</h3>';
                html += '<div id="debugInfo" class="text-sm text-gray-600 max-h-96 overflow-y-auto hidden bg-white p-3 rounded border">';
                results.debug.forEach(debug => {
                    html += `<p class="font-mono text-xs mb-1 py-1 border-b border-gray-100">• ${debug}</p>`;
                });
                html += '</div>';
                html += '</div>';
            }

            html += '</div>';
            resultsDiv.innerHTML = html;
            
            resultsDiv.scrollIntoView({ behavior: 'smooth' });
        }

        function renderDiscrepancy(disc, color) {
            let html = `<div class="bg-${color}-50 border-l-4 border-${color}-400 rounded-lg p-4 mb-4 shadow-sm">`;
            html += '<div class="flex justify-between items-start mb-3">';
            html += `<strong class="text-lg text-${color}-800">SKU: ${disc.sku}</strong>`;
            html += '<div class="text-right">';
            if (disc.matchedAs !== disc.sku) {
                html += `<span class="text-xs bg-${color}-200 px-2 py-1 rounded mr-2">Encontrado como: ${disc.matchedAs}</span>`;
            }
            html += `<span class="text-xs bg-gray-200 px-2 py-1 rounded">Puntuación: ${disc.totalScore}%</span>`;
            html += '</div>';
            html += '</div>';
            
            // Detailed scores
            if (disc.scores) {
                html += '<div class="mb-4 p-3 bg-gray-50 rounded-lg">';
                html += '<h4 class="font-semibold text-gray-700 mb-2">📊 Puntuaciones Detalladas:</h4>';
                html += '<div class="grid grid-cols-3 gap-3 text-sm">';
                html += `<div class="text-center p-2 bg-white rounded">
                            <div class="font-bold text-lg ${disc.scores.nombre >= 40 ? 'text-green-600' : 'text-red-600'}">${disc.scores.nombre.toFixed(1)}%</div>
                            <div class="text-xs text-gray-600">Nombre</div>
                         </div>`;
                html += `<div class="text-center p-2 bg-white rounded">
                            <div class="font-bold text-lg ${disc.scores.precio >= 90 ? 'text-green-600' : 'text-red-600'}">${disc.scores.precio.toFixed(1)}%</div>
                            <div class="text-xs text-gray-600">Precio</div>
                         </div>`;
                html += `<div class="text-center p-2 bg-white rounded">
                            <div class="font-bold text-lg ${disc.scores.descripcion >= 25 ? 'text-green-600' : 'text-red-600'}">${disc.scores.descripcion.toFixed(1)}%</div>
                            <div class="text-xs text-gray-600">Descripción</div>
                         </div>`;
                html += '</div>';
                html += '</div>';
            }
            
            html += '<div class="grid md:grid-cols-2 gap-4 mb-4">';
            html += '<div>';
            html += '<h4 class="font-semibold text-gray-700 mb-2">📋 Datos del Excel:</h4>';
            html += '<div class="bg-white p-3 rounded-lg border text-sm">';
            html += `<p><strong>Nombre:</strong> ${disc.expected.nombre || 'N/A'}</p>`;
            html += `<p><strong>Precio:</strong> ${disc.expected.precio || 'N/A'}</p>`;
            html += `<p><strong>Descripción:</strong> ${disc.expected.descripcion || 'N/A'}</p>`;
            html += '</div>';
            html += '</div>';
            
            html += '<div>';
            html += '<h4 class="font-semibold text-gray-700 mb-2">📄 Encontrado en PDF:</h4>';
            html += '<div class="bg-white p-3 rounded-lg border text-sm max-h-32 overflow-y-auto">';
            html += disc.found.substring(0, 500) + (disc.found.length > 500 ? '...' : '');
            html += '</div>';
            html += '</div>';
            html += '</div>';
            
            html += `<div class="p-3 bg-${color}-100 rounded-lg">`;
            html += `<strong class="text-${color}-700">🚨 Problemas Detectados:</strong><br>`;
            disc.issues.forEach(issue => {
                html += `<div class="text-${color}-600 text-sm mt-1 pl-2 border-l-2 border-${color}-300">• ${issue}</div>`;
            });
            html += '</div>';
            
            html += '</div>';
            
            return html;
        }

        function toggleDebug() {
            const debugInfo = document.getElementById('debugInfo');
            debugInfo.classList.toggle('hidden');
        }

        // Initialize when page loads
        document.addEventListener('DOMContentLoaded', setupFileHandlers);
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'96a06a4f502a32cd',t:'MTc1NDMzNTI2MC4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
