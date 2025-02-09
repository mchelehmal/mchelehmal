<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TebSin.ai - Recording Page</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 50px;
        }
        #output {
            width: 80%;
            height: 200px;
            margin-top: 20px;
            padding: 10px;
            border: 1px solid #ccc;
        }
        button {
            margin: 10px;
            padding: 10px;
            font-size: 16px;
        }
    </style>
</head>
<body>
    <h1>TebSin.ai</h1>
    <h2>AI-Powered Medical Transcription</h2>
    
    <button id="record">Start Recording</button>
    <button id="pause" disabled>Pause</button>
    <button id="stop" disabled>Stop</button>
    
    <h3>Transcribed Text:</h3>
    <textarea id="output" readonly></textarea>
    <br>
    <button id="copy">Copy to Clipboard</button>
    
    <script>
        let mediaRecorder;
        let audioChunks = [];
        let isPaused = false;
        
        document.getElementById("record").addEventListener("click", async () => {
            const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
            mediaRecorder = new MediaRecorder(stream);
            
            mediaRecorder.ondataavailable = event => {
                audioChunks.push(event.data);
            };
            
            mediaRecorder.onstop = async () => {
    const audioBlob = new Blob(audioChunks, { type: 'audio/wav' });
    const fileName = `recording_${Date.now()}.wav`;

    const formData = new FormData();
    formData.append("audio", audioBlob, fileName);

    // Upload to AWS S3
    fetch("https://6wu6af3xok.execute-api.us-east-1.amazonaws.com", {
        method: "POST",
        body: formData,
    })
    .then(response => response.json())
    .then(data => {
        console.log("Uploaded:", data);
        startTranscription(fileName);
    })
    .catch(error => console.error("Error uploading file:", error));
};

function startTranscription(fileName) {
    fetch(`https://https://6wu6af3xok.execute-api.us-east-1.amazonaws.com/transcribe?file_name=${fileName}`)
        .then(response => response.json())
        .then(data => {
            document.getElementById("output").value = "Transcription is processing...";
        })
        .catch(error => console.error("Error starting transcription:", error));
}

            
            mediaRecorder.start();
            document.getElementById("record").disabled = true;
            document.getElementById("pause").disabled = false;
            document.getElementById("stop").disabled = false;
        });
        
        document.getElementById("pause").addEventListener("click", () => {
            if (mediaRecorder.state === "recording") {
                mediaRecorder.pause();
                document.getElementById("pause").textContent = "Resume";
            } else {
                mediaRecorder.resume();
                document.getElementById("pause").textContent = "Pause";
            }
        });
        
        document.getElementById("stop").addEventListener("click", () => {
            mediaRecorder.stop();
            document.getElementById("record").disabled = false;
            document.getElementById("pause").disabled = true;
            document.getElementById("stop").disabled = true;
        });
        
        document.getElementById("copy").addEventListener("click", () => {
            const outputText = document.getElementById("output");
            outputText.select();
            document.execCommand("copy");
            alert("Copied to clipboard!");
        });
    </script>
</body>
</html>
