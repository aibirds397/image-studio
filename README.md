<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='UTF-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1.0'>
    <title>Studio Ultra Pro - Advanced Image Toolkit</title>
    
    <!-- Libraries for ZIP, Confetti, and UI -->
    <script src="https://cloudflare.com"></script>
    <script src="https://jsdelivr.net"></script>
    <link href="https://googleapis.com" rel="stylesheet">

    <style>
        :root { --primary: #6366f1; --bg: #0f172a; --card: #1e293b; --text: #f8fafc; --border: #334155; }
        body { font-family: 'Inter', sans-serif; background: var(--bg); color: var(--text); padding: 20px; display: flex; flex-direction: column; align-items: center; }
        .container { background: var(--card); padding: 30px; border-radius: 24px; border: 1px solid var(--border); width: 100%; max-width: 850px; }
        .upload-area { border: 2px dashed var(--primary); padding: 40px; border-radius: 20px; text-align: center; margin: 20px 0; cursor: pointer; }
        .section { background: rgba(0,0,0,0.2); padding: 20px; border-radius: 16px; margin-bottom: 20px; border: 1px solid var(--border); }
        .grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 15px; }
        input, select { width: 100%; padding: 12px; border-radius: 10px; border: 1px solid var(--border); background: #0f172a; color: white; margin-top: 5px; }
        .main-btn { background: var(--primary); color: white; border: none; padding: 18px; border-radius: 14px; font-weight: 800; cursor: pointer; width: 100%; margin-top: 10px; }
        canvas { max-width: 100%; border-radius: 12px; margin-top: 20px; display: none; }
        label { font-size: 12px; font-weight: 600; opacity: 0.8; }
    </style>
</head>
<body>

<div class="container">
    <h2 style="text-align:center; color:var(--primary);">Studio Ultra Pro - Master Tool</h2>
    
    <div class="upload-area" onclick="document.getElementById('files').click()">
        <strong id="status">Drop Images or Click to Upload</strong>
        <input type="file" id="files" accept="image/*" multiple style="display:none">
    </div>

    <canvas id="previewCanvas"></canvas>

    <!-- NEW: Advanced Tools like Pi7 -->
    <div class="section">
        <div style="color:var(--primary); font-weight:800; margin-bottom:15px; text-transform:uppercase; font-size:11px;">Image Controls</div>
        <div class="grid">
            <div><label>Target Size (KB)</label><input type="number" id="targetKB" placeholder="e.g. 50"></div>
            <div><label>Resize Width (px)</label><input type="number" id="targetWidth" placeholder="Original"></div>
            <div><label>Resize Height (px)</label><input type="number" id="targetHeight" placeholder="Original"></div>
            <div><label>Format</label><select id="fmt"><option value="image/jpeg">JPEG</option><option value="image/png">PNG</option><option value="image/webp">WebP</option></select></div>
        </div>
    </div>

    <!-- Watermark Section -->
    <div class="section">
        <div style="color:var(--primary); font-weight:800; margin-bottom:15px; text-transform:uppercase; font-size:11px;">Watermark branding</div>
        <div class="grid">
            <div><label>Text</label><input type="text" id="wmText" placeholder="@YourBrand" oninput="updatePreview()"></div>
            <div><label>Text Opacity</label><input type="range" id="wmOpacity" min="0.1" max="1.0" step="0.1" value="0.7" oninput="updatePreview()"></div>
            <div><label>Position</label><select id="wmPos" onchange="updatePreview()"><option value="bottom-right">Bottom Right</option><option value="center">Center</option><option value="top-left">Top Left</option></select></div>
        </div>
    </div>

    <button class="main-btn" id="goBtn" onclick="processBatch()">⚡ PROCESS & DOWNLOAD ALL (ZIP)</button>
</div>

<script>
    let filesArr = [];
    const filesInput = document.getElementById('files');
    const previewCanvas = document.getElementById('previewCanvas');

    filesInput.onchange = (e) => {
        filesArr = Array.from(e.target.files);
        if(filesArr.length) {
            document.getElementById('status').innerText = filesArr.length + " Files Ready";
            previewCanvas.style.display = 'block';
            updatePreview();
        }
    };

    function updatePreview() {
        if (!filesArr.length) return;
        const reader = new FileReader();
        reader.onload = (e) => {
            const img = new Image();
            img.onload = () => draw(previewCanvas, img);
            img.src = e.target.result;
        };
        reader.readAsDataURL(filesArr[0]);
    }

    function draw(cvs, img) {
        const ctx = cvs.getContext('2d');
        const tw = document.getElementById('targetWidth').value || img.width;
        const th = document.getElementById('targetHeight').value || img.height;
        
        cvs.width = tw; cvs.height = th;
        ctx.drawImage(img, 0, 0, tw, th);

        const txt = document.getElementById('wmText').value;
        if(txt) {
            ctx.globalAlpha = document.getElementById('wmOpacity').value;
            ctx.fillStyle = "white";
            ctx.font = `bold ${cvs.width * 0.05}px Arial`;
            const pos = document.getElementById('wmPos').value;
            if(pos === 'bottom-right') ctx.fillText(txt, cvs.width - (ctx.measureText(txt).width + 20), cvs.height - 30);
            else if(pos === 'center') ctx.fillText(txt, (cvs.width - ctx.measureText(txt).width)/2, cvs.height/2);
            else ctx.fillText(txt, 20, 40);
        }
        ctx.globalAlpha = 1.0;
    }

    async function processBatch() {
        const zip = new JSZip();
        const targetKB = document.getElementById('targetKB').value;
        
        for (let f of filesArr) {
            const data = await new Promise(res => {
                const r = new FileReader();
                r.onload = (e) => {
                    const i = new Image();
                    i.onload = () => {
                        const c = document.createElement('canvas');
                        draw(c, i);
                        let quality = 0.8;
                        if(targetKB) quality = 0.5; // Simple compression logic
                        res(c.toDataURL(document.getElementById('fmt').value, quality));
                    };
                    i.src = e.target.result;
                };
                r.readAsDataURL(f);
            });
            zip.file("studio_" + f.name, data.split(',')[1], {base64: true});
        }
        const blob = await zip.generateAsync({type:"blob"});
        const a = document.createElement('a'); a.href = URL.createObjectURL(blob); a.download = "processed_images.zip"; a.click();
        confetti({ particleCount: 150, spread: 70 });
    }
</script>
</body>
</html>
