🌿 PlantAI — Medicinal Plant Recognition System

A Python desktop app that identifies medicinal plants from a photo using transfer learning (MobileNetV2 + TensorFlow/Keras), then shows the plant's family, category, description, traditionally reported uses and safety precautions from a local knowledge base. It also keeps a history and statistics of every recognition.

Everything runs offline — no API key or internet connection is needed once the dependencies are installed.

⚠️ Disclaimer: AI identification does not confirm that a plant is safe to consume or suitable for treating any condition. The health information in this app comes from a local, editable reference file (data/plants_data.json) and is for general educational purposes only. It is not medical advice. Consult a qualified professional before any medicinal use.

✨ Features
Feature	Description
📷 Image upload	Pick a plant/leaf photo (jpg, png, bmp, webp) with a clear preview
🤖 AI recognition	MobileNetV2 transfer-learning classifier for 15 plants
📊 Confidence score	Confidence % plus the top-3 alternative predictions
🌱 Structured plant details	Family, category, parts used, description, traditional uses, each in its own labelled section
⚠️ Safety info	Precautions highlighted in a separate section for every plant
🔍 Search & filter	Search by name, scientific name, family or use; filter by category
📚 Plant library	Browse all supported plants without needing a photo
⭐ Favorites	Star plants for quick access
📜 History	Every prediction is timestamped and saved locally
📤 CSV export	Export your history to a spreadsheet
💾 Reports	Save a .txt and a styled .html report per recognition
📈 Statistics	Totals, most-recognized plants, category breakdown
🤖 AI Model tab	Training accuracy curves and model info
🌓 Dark / Light mode	Toggle from the top bar
🌿 Supported plants

Aloe Vera · Amla · Arjuna · Ashwagandha · Brahmi · Curry Leaf · Giloy · Ginger · Lemongrass · Mint · Moringa · Neem · Shatavari · Tulsi · Turmeric

🧠 How the model works

The classifier uses MobileNetV2 pretrained on ImageNet as a frozen feature extractor, with a small trainable head on top:

Input (128×128 RGB, scaled to [0, 1])
  → Rescaling to [-1, 1]
  → MobileNetV2 (frozen, ImageNet weights)
  → GlobalAveragePooling
  → Dense(128, ReLU) → Dropout(0.3)
  → Dense(15, Softmax)

Why transfer learning? A CNN trained from scratch on only ~20 photos per plant memorizes the training images and fails on new ones (validation accuracy stays near random). MobileNetV2 already understands edges, textures and shapes, so only the small head has to be learned, which works far better on small datasets.

Training uses an 80/20 train/validation split, light augmentation (rotation, zoom, flips), and early stopping that keeps the best weights.

🗂️ Project structure
PlantAI/
├── main.py                  # App entry point — run this
├── config.py                # Paths, image size, app constants
├── requirements.txt
│
├── data/
│   ├── plants_data.json     # Editable plant knowledge base
│   ├── history.json         # Auto-created: recognition history
│   └── favorites.json       # Auto-created: starred plants
│
├── model/
│   ├── model_builder.py     # MobileNetV2 transfer-learning model
│   ├── train.py             # Trains on the dataset/ folder
│   ├── predict.py           # Loads the model and predicts
│   ├── plant_model.h5       # Created by training
│   ├── class_indices.json   # Created by training
│   └── training_history.json
│
├── utils/
│   ├── database.py          # JSON-backed history, favorites, search, stats
│   ├── image_utils.py       # OpenCV/Pillow preprocessing and thumbnails
│   └── report_utils.py      # .txt / .html report generation
│
├── gui/
│   ├── app.py               # Tkinter window and all 6 tabs
│   ├── theme.py             # Light/Dark themes
│   └── charts.py            # Dependency-free canvas charts
│
├── dataset/                 # Training images, one folder per plant
├── reports/                 # Saved reports (git-ignored)
└── sample_images/           # Test images
⚙️ Installation (Windows / PowerShell)

Python 3.11 is recommended (this is the version the project was tested with). TensorFlow does not support every newer Python release.

powershell
git clone https://github.com/VivekChahal24/PlantAI.git
cd PlantAI

py -3.11 -m venv venv
.\venv\Scripts\Activate.ps1

python -m pip install --upgrade pip
python -m pip install -r requirements.txt

💡 Always use python -m pip install ... (not just pip) so packages go into the same Python that runs the app. Mixing Python versions inside one virtual environment causes errors like numpy._core._multiarray_umath or cannot import name '_imaging'. If that happens, delete venv and recreate it.

On Linux, if Tkinter is missing: sudo apt-get install python3-tk

🚀 Run the app
powershell
python main.py

The Library, Favorites, History and Statistics tabs work immediately. The Recognize tab needs a trained model (below).

🏋️ Training the model

The repository does not include the plant photos, so you add your own:

Put photos of each plant in its folder under dataset/:
   dataset/
     Tulsi/     <- Tulsi photos
     Neem/      <- Neem photos
     ...
Make sure every file is a real image (.jpg, .png). Pages saved from a browser as .htm or broken downloads will be ignored or cause errors.
Train:
powershell
   python model/train.py

The first run needs internet once, to download the pretrained MobileNetV2 weights (~10–14 MB). 4. Restart the app. The header shows ✅ Model loaded when it is ready.

How many photos? With transfer learning, 20–30 per plant gives a working model, but 50–100+ varied photos per plant (different angles, lighting, backgrounds) improves accuracy noticeably.

Adding a new plant
Create dataset/<PlantName>/ and add photos.
Add a matching entry to data/plants_data.json (scientific_name, family, category, description, parts_used, traditional_uses, precautions).
Run python model/train.py again.
📄 Reports

After a prediction, click Save Report to write reports/<Plant>_<timestamp>.txt and .html (the HTML report embeds the uploaded image). Use History → Export CSV for a spreadsheet of all recognitions.

⚠️ Known limitations
Accuracy depends heavily on the size and quality of your training dataset. Plants that look alike (for example Tulsi and Mint) can be confused when there are few photos.
Photos with a clear, close-up view of the leaf or plant part on a plain background give the best results.
The model only knows the 15 plants it was trained on; any other plant will be mapped to the closest of those.
🧩 Tech stack

TensorFlow / Keras · OpenCV · NumPy · Pandas · Pillow · Tkinter

🛣️ Ideas for further work
Add a confusion matrix view to the AI Model tab
Fine-tune the top MobileNetV2 layers once more data is available
Add a "Plant of the Day" widget
Package as a one-click .exe with PyInstaller