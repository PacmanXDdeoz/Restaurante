<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>JavaScript PDF Viewer</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            height: 100%;
            width: 100%;
            overflow: hidden;
            font-family: Arial, sans-serif;
        }
        
        #pdf-container {
            width: 100%;
            height: 100vh;
            overflow: hidden;
            background-color: #525659;
            position: relative;
        }
        
        #pdf-viewer {
            width: 100%;
            height: 100%;
            overflow: auto;
            display: flex;
            justify-content: center;
        }
        
        #pdf-canvas {
            margin: 0 auto;
            display: block;
        }
        
        #loading-message {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: white;
            font-size: 18px;
        }
        
        #controls {
            position: fixed;
            bottom: 0;
            left: 0;
            right: 0;
            background-color: rgba(0, 0, 0, 0.7);
            padding: 10px;
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 100;
        }
        
        #controls button {
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 4px;
            padding: 8px 15px;
            margin: 0 5px;
            cursor: pointer;
            font-size: 14px;
        }
        
        #controls button:disabled {
            background-color: #cccccc;
            cursor: not-allowed;
        }
        
        #page-num {
            color: white;
            margin: 0 15px;
        }
        
        #zoom-controls {
            margin-left: 20px;
        }
        
        @media screen and (max-width: 768px) {
            #controls {
                padding: 8px 5px;
            }
            
            #controls button {
                padding: 6px 10px;
                font-size: 12px;
            }
            
            #page-num {
                margin: 0 10px;
                font-size: 12px;
            }
        }
    </style>
</head>
<body>
    <div id="pdf-container">
        <div id="loading-message">Loading PDF...</div>
        <div id="pdf-viewer">
            <canvas id="pdf-canvas"></canvas>
        </div>
        <div id="controls">
            <button id="prev-page">Previous</button>
            <span id="page-num">Page 1 of 1</span>
            <button id="next-page">Next</button>
            <div id="zoom-controls">
                <button id="zoom-out">-</button>
                <button id="zoom-in">+</button>
            </div>
        </div>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.4.120/pdf.min.js"></script>
    <script>
        pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.4.120/pdf.worker.min.js';
        
        const pdfFile = 'Menu.pdf';

        let pdfDoc = null;
        let pageNum = 1;
        let pageRendering = false;
        let pageNumPending = null;
        let scale = 1.0;
        const canvas = document.getElementById('pdf-canvas');
        const ctx = canvas.getContext('2d');
        
        function renderPage(num) {
            pageRendering = true;
            
            document.getElementById('loading-message').style.display = 'block';
            
            pdfDoc.getPage(num).then(function(page) {
                const viewport = page.getViewport({ scale: scale });
                canvas.height = viewport.height;
                canvas.width = viewport.width;
                
                const renderContext = {
                    canvasContext: ctx,
                    viewport: viewport
                };
                
                const renderTask = page.render(renderContext);
                
                renderTask.promise.then(function() {
                    pageRendering = false;
                    document.getElementById('loading-message').style.display = 'none';
                    
                    if (pageNumPending !== null) {
                        renderPage(pageNumPending);
                        pageNumPending = null;
                    }
                });
            });
            
            document.getElementById('page-num').textContent = `Page ${num} of ${pdfDoc.numPages}`;
            
            document.getElementById('prev-page').disabled = num <= 1;
            document.getElementById('next-page').disabled = num >= pdfDoc.numPages;
        }
        
        function queueRenderPage(num) {
            if (pageRendering) {
                pageNumPending = num;
            } else {
                renderPage(num);
            }
        }
        
        function onPrevPage() {
            if (pageNum <= 1) {
                return;
            }
            pageNum--;
            queueRenderPage(pageNum);
        }
        
        function onNextPage() {
            if (pageNum >= pdfDoc.numPages) {
                return;
            }
            pageNum++;
            queueRenderPage(pageNum);
        }
        
        function zoomIn() {
            scale += 0.25;
            queueRenderPage(pageNum);
        }
        
        function zoomOut() {
            if (scale <= 0.5) return;
            scale -= 0.25;
            queueRenderPage(pageNum);
        }
        

        function initPdfViewer() {
            pdfjsLib.getDocument(pdfFile).promise.then(function(pdf) {
                pdfDoc = pdf;
                document.getElementById('page-num').textContent = `Page ${pageNum} of ${pdf.numPages}`;
                
                renderPage(pageNum);
            }).catch(function(error) {
                console.error('Error loading PDF:', error);
                document.getElementById('loading-message').textContent = 'Error loading PDF. Please try again.';
            });
            
            document.getElementById('prev-page').addEventListener('click', onPrevPage);
            document.getElementById('next-page').addEventListener('click', onNextPage);
            document.getElementById('zoom-in').addEventListener('click', zoomIn);
            document.getElementById('zoom-out').addEventListener('click', zoomOut);
            
            let touchStartX = 0;
            let touchEndX = 0;
            
            document.getElementById('pdf-viewer').addEventListener('touchstart', function(e) {
                touchStartX = e.changedTouches[0].screenX;
            });
            
            document.getElementById('pdf-viewer').addEventListener('touchend', function(e) {
                touchEndX = e.changedTouches[0].screenX;
                handleSwipe();
            });
            
            function handleSwipe() {
                const swipeThreshold = 50;
                if (touchEndX < touchStartX - swipeThreshold) {
                    onNextPage();
                } else if (touchEndX > touchStartX + swipeThreshold) {
                    onPrevPage();
                }
            }
        }
        
        // Initialize when the page loads
        document.addEventListener('DOMContentLoaded', initPdfViewer);
    </script>
</body>
</html>