<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Menu</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            height: 100%;
            width: 100%;
            overflow: hidden;
            font-family: Arial, sans-serif;
            background-color: #525659;
        }
        
        #pdf-container {
            width: 100%;
            height: 100vh;
            overflow: hidden;
            position: relative;
            display: flex;
            flex-direction: column;
        }
        
        #pdf-viewer {
            flex: 1;
            overflow: auto;
            display: flex;
            justify-content: center;
            align-items: flex-start;
            -webkit-overflow-scrolling: touch; /* For smooth scrolling on iOS */
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
            background-color: rgba(0, 0, 0, 0.7);
            padding: 15px 20px;
            border-radius: 5px;
            z-index: 200;
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
            margin-left: 10px;
        }
        
        @media screen and (max-width: 768px) {
            #controls {
                padding: 8px 5px;
                flex-wrap: wrap;
                justify-content: space-around;
            }
            
            #controls button {
                padding: 6px 10px;
                font-size: 12px;
                margin: 2px;
            }
            
            #page-num {
                margin: 0 5px;
                font-size: 12px;
                order: 3;
                width: 100%;
                text-align: center;
                margin-top: 5px;
            }
            
            #zoom-controls {
                margin-left: 0;
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
            <button id="next-page">Next</button>
            <span id="page-num">Page 1 of 1</span>
            <div id="zoom-controls">
                <button id="zoom-out">-</button>
                <button id="fit-width">Fit Width</button>
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
        let originalScale = 1.0;
        const canvas = document.getElementById('pdf-canvas');
        const ctx = canvas.getContext('2d');
        const pdfViewer = document.getElementById('pdf-viewer');
        
        const isMobile = window.innerWidth <= 768;
        
        function calculateFitToWidthScale(page) {
            const viewport = page.getViewport({ scale: 1.0 });
            const containerWidth = pdfViewer.clientWidth;
            // Add padding to prevent touching the edges
            const padding = isMobile ? 10 : 20;
            return (containerWidth - padding) / viewport.width;
        }
        
        function renderPage(num) {
            pageRendering = true;
            
            document.getElementById('loading-message').style.display = 'block';
            
            pdfDoc.getPage(num).then(function(page) {
                if (num === 1 && pageNumPending === null) {
                    if (isMobile) {
                        scale = calculateFitToWidthScale(page);
                        originalScale = scale;
                    }
                }
                
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
            scale += scale * 0.2;
            queueRenderPage(pageNum);
        }

        function zoomOut() {
            if (scale <= 0.5) return;
            scale -= scale * 0.2;
            queueRenderPage(pageNum);
        }

        function fitToWidth() {
            pdfDoc.getPage(pageNum).then(function(page) {
                scale = calculateFitToWidthScale(page);
                queueRenderPage(pageNum);
            });
        }

        function handleResize() {
            if (document.getElementById('fit-width').classList.contains('active')) {
                fitToWidth();
            }
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
            document.getElementById('fit-width').addEventListener('click', fitToWidth);

            if (isMobile) {
                document.getElementById('fit-width').classList.add('active');
            }
            
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
            
            window.addEventListener('resize', handleResize);
            
            document.addEventListener('keydown', function(e) {
                if (e.key === 'ArrowRight' || e.key === ' ') {
                    onNextPage();
                } else if (e.key === 'ArrowLeft') {
                    onPrevPage();
                }
            });
        }
        
        document.addEventListener('DOMContentLoaded', initPdfViewer);
    </script>
</body>
</html>