<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>亚马逊 PC端 高级预览工具</title>
    <style>
        :root {
            --amazon-orange: #ff9900;
            --amazon-dark: #232f3e;
            --bg-color: #f2f2f2;
            --border-color: #ccc;
        }
        body {
            font-family: Arial, sans-serif;
            background-color: var(--bg-color);
            margin: 0;
            padding: 20px;
            color: #333;
        }
        .container {
            max-width: 1300px;
            margin: 0 auto;
            background: #fff;
            padding: 20px 40px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
            border-radius: 8px;
        }
        h1 {
            color: var(--amazon-dark);
            text-align: center;
            border-bottom: 2px solid var(--amazon-orange);
            padding-bottom: 10px;
        }
        .section-title {
            background: var(--amazon-dark);
            color: #fff;
            padding: 10px 15px;
            border-radius: 4px;
            margin-top: 40px;
        }
        .tips {
            font-size: 13px;
            color: #666;
            margin-top: 5px;
            margin-bottom: 15px;
        }

        /* --- 套图区域样式 (缩略图 + 主图) --- */
        .gallery-layout {
            display: flex;
            gap: 20px;
            margin-top: 20px;
        }
        .gallery-upload-box {
            margin-bottom: 20px;
        }
        .thumbnails-container {
            display: flex;
            flex-direction: column;
            gap: 10px;
            width: 60px;
        }
        .thumb-item {
            width: 60px;
            height: 60px;
            border: 1px solid var(--border-color);
            border-radius: 4px;
            cursor: pointer;
            overflow: hidden;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .thumb-item:hover {
            border-color: #999;
        }
        .thumb-item.active {
            border: 2px solid var(--amazon-orange);
            box-shadow: 0 0 5px rgba(255, 153, 0, 0.5);
        }
        .thumb-item img {
            max-width: 100%;
            max-height: 100%;
            object-fit: contain;
        }
        .main-image-container {
            width: 500px;
            height: 500px;
            border: 1px solid #eee;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .main-image-container img {
            max-width: 100%;
            max-height: 100%;
            object-fit: contain;
        }

        /* --- A+ 区域样式 (左侧控制 + 右侧无缝预览) --- */
        .aplus-layout {
            display: flex;
            gap: 30px;
            margin-top: 20px;
            align-items: flex-start;
        }
        .aplus-controls {
            width: 220px;
            display: flex;
            flex-direction: column;
            gap: 15px;
        }
        .control-box {
            background: #f9f9f9;
            border: 1px solid #ddd;
            padding: 10px;
            border-radius: 4px;
        }
        .control-box label {
            font-weight: bold;
            display: block;
            margin-bottom: 5px;
            color: var(--amazon-dark);
        }
        .control-box input {
            width: 100%;
        }

        .aplus-preview {
            width: 970px;
            display: flex;
            flex-direction: column;
            /* 关键：消除图片之间的空隙 */
            gap: 0; 
        }

        .aplus-module {
            width: 970px;
            position: relative;
            /* 关键：消除行内元素底部的留白 */
            line-height: 0; 
            font-size: 0;
        }

        .scroll-container {
            display: flex;
            overflow-x: hidden; /* 隐藏原生滚动条，完全依赖箭头 */
            scroll-snap-type: x mandatory;
            width: 970px;
            scroll-behavior: smooth;
        }
        .scroll-container img {
            width: 970px;
            flex-shrink: 0;
            scroll-snap-align: start;
            display: block; /* 消除底部留白 */
        }

        /* 切换箭头样式 */
        .arrow-btn {
            position: absolute;
            top: 50%;
            transform: translateY(-50%);
            background: rgba(255, 255, 255, 0.9);
            border: 1px solid #ccc;
            border-radius: 4px;
            cursor: pointer;
            width: 40px;
            height: 60px;
            font-size: 24px;
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 10;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
            display: none; /* 默认隐藏，有多张图时才显示 */
        }
        .arrow-btn:hover {
            background: #fff;
            border-color: #999;
        }
        .arrow-left { left: 10px; }
        .arrow-right { right: 10px; }

    </style>
</head>
<body>

<div class="container">
    <h1>亚马逊 PC端 高级预览工具</h1>
    
    <h2 class="section-title">1. 产品套图 (左侧缩略图 + 右侧主图展示)</h2>
    <div class="gallery-upload-box">
        <input type="file" id="gallery-upload" accept="image/*" multiple>
        <p class="tips">提示：按住 Ctrl/Command 多选，上限8张。点击左侧缩略图可切换右侧主视图。</p>
    </div>
    
    <div class="gallery-layout">
        <div class="thumbnails-container" id="gallery-thumbs"></div>
        <div class="main-image-container" id="gallery-main">
            <span style="color:#aaa;">上传图片后在此预览</span>
        </div>
    </div>

    <h2 class="section-title">2. A+ 页面 (无缝拼接 + 箭头切换)</h2>
    <p class="tips">提示：在左侧分别上传对应板块的图片。上传多张即可激活左右切换箭头。右侧视图为 970px 无缝拼接，还原真实买家视角。</p>
    
    <div class="aplus-layout">
        <div class="aplus-controls" id="aplus-controls">
            </div>

        <div class="aplus-preview" id="aplus-preview">
            </div>
    </div>
</div>

<script>
    // ================= 1. 套图逻辑 (缩略图 + 大图) =================
    document.getElementById('gallery-upload').addEventListener('change', function(event) {
        const thumbsContainer = document.getElementById('gallery-thumbs');
        const mainContainer = document.getElementById('gallery-main');
        thumbsContainer.innerHTML = ''; 
        mainContainer.innerHTML = ''; 
        
        let files = Array.from(event.target.files);
        if (files.length > 8) {
            alert('套图最多只能上传 8 张，超出部分已被自动忽略。');
            files = files.slice(0, 8);
        }

        files.forEach((file, index) => {
            if (!file.type.startsWith('image/')) return;
            const reader = new FileReader();
            reader.onload = function(e) {
                // 生成缩略图
                const thumbDiv = document.createElement('div');
                thumbDiv.className = 'thumb-item';
                if (index === 0) thumbDiv.classList.add('active'); // 默认选中第一张
                
                const thumbImg = document.createElement('img');
                thumbImg.src = e.target.result;
                thumbDiv.appendChild(thumbImg);
                
                // 点击缩略图事件
                thumbDiv.onclick = function() {
                    // 移除所有高亮
                    document.querySelectorAll('.thumb-item').forEach(el => el.classList.remove('active'));
                    // 添加当前高亮
                    this.classList.add('active');
                    // 替换主图
                    mainContainer.innerHTML = `<img src="${e.target.result}" alt="主图预览">`;
                };

                thumbsContainer.appendChild(thumbDiv);

                // 默认将第一张图设为主图
                if (index === 0) {
                    mainContainer.innerHTML = `<img src="${e.target.result}" alt="主图预览">`;
                }
            }
            reader.readAsDataURL(file);
        });
    });

    // ================= 2. A+ 逻辑 (左侧控制 + 无缝展示 + 箭头) =================
    const aplusControls = document.getElementById('aplus-controls');
    const aplusPreview = document.getElementById('aplus-preview');
    
    for (let i = 1; i <= 7; i++) {
        // --- 构建左侧控制面板 ---
        const controlBox = document.createElement('div');
        controlBox.className = 'control-box';
        controlBox.innerHTML = `
            <label>板块 ${i}</label>
            <input type="file" accept="image/*" multiple id="aplus-upload-${i}">
        `;
        aplusControls.appendChild(controlBox);

        // --- 构建右侧展示面板 ---
        const moduleBox = document.createElement('div');
        moduleBox.className = 'aplus-module';
        
        // 滚动容器
        const scrollContainer = document.createElement('div');
        scrollContainer.className = 'scroll-container';
        scrollContainer.id = `aplus-scroll-${i}`;
        
        // 左右箭头
        const btnLeft = document.createElement('div');
        btnLeft.className = 'arrow-btn arrow-left';
        btnLeft.innerHTML = '&#10094;'; // 左箭头符号
        
        const btnRight = document.createElement('div');
        btnRight.className = 'arrow-btn arrow-right';
        btnRight.innerHTML = '&#10095;'; // 右箭头符号

        moduleBox.appendChild(btnLeft);
        moduleBox.appendChild(scrollContainer);
        moduleBox.appendChild(btnRight);
        aplusPreview.appendChild(moduleBox);

        // --- 箭头点击滚动事件 ---
        btnLeft.onclick = () => {
            scrollContainer.scrollBy({ left: -970, behavior: 'smooth' });
        };
        btnRight.onclick = () => {
            scrollContainer.scrollBy({ left: 970, behavior: 'smooth' });
        };

        // --- 绑定上传事件 ---
        document.getElementById(`aplus-upload-${i}`).addEventListener('change', function(event) {
            scrollContainer.innerHTML = ''; // 清空
            const files = event.target.files;
            
            // 判断是否显示箭头 (多于1张图才显示)
            if (files.length > 1) {
                btnLeft.style.display = 'flex';
                btnRight.style.display = 'flex';
            } else {
                btnLeft.style.display = 'none';
                btnRight.style.display = 'none';
            }

            for (let j = 0; j < files.length; j++) {
                const file = files[j];
                if (!file.type.startsWith('image/')) continue;
                
                const reader = new FileReader();
                reader.onload = function(e) {
                    const img = document.createElement('img');
                    img.src = e.target.result;
                    scrollContainer.appendChild(img);
                }
                reader.readAsDataURL(file);
            }
        });
    }
</script>

</body>
</html>
