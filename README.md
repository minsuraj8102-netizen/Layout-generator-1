from flask import Flask, render_template, request, send_file
import cv2
import os

app = Flask(__name__)
UPLOAD_FOLDER = "uploads"
GENERATED_FOLDER = "generated"
os.makedirs(UPLOAD_FOLDER, exist_ok=True)
os.makedirs(GENERATED_FOLDER, exist_ok=True)

@app.route("/", methods=["GET", "POST"])
def index():
    if request.method == "POST":
        file = request.files["image"]
        filepath = os.path.join(UPLOAD_FOLDER, file.filename)
        file.save(filepath)

        html_code = generate_html(filepath)
        generated_file = os.path.join(GENERATED_FOLDER, "output.html")
        with open(generated_file, "w") as f:
            f.write(html_code)

        return send_file(generated_file, as_attachment=True)

    return render_template("index.html")

def generate_html(image_path):
    # Load image
    img = cv2.imread(image_path)
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    _, thresh = cv2.threshold(gray, 200, 255, cv2.THRESH_BINARY_INV)

    contours, _ = cv2.findContours(thresh, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

    html_elements = ""
    for i, cnt in enumerate(contours):
        x, y, w, h = cv2.boundingRect(cnt)
        html_elements += f'<div style="position:absolute; left:{x}px; top:{y}px; width:{w}px; height:{h}px; border:1px solid #000;"></div>\n'

    html_code = f"""
    <!DOCTYPE html>
    <html>
    <head>
    <title>Generated Layout</title>
    <style>
    body {{ position: relative; width: {img.shape[1]}px; height: {img.shape[0]}px; margin: 0; }}
    </style>
    </head>
    <body>
    {html_elements}
    </body>
    </html>
    """
    return html_code

if __name__ == "__main__":
    app.run(debug=True)
