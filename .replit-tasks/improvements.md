# Face Consistency Pipeline — Replit Agent Build Spec

Build from scratch. Commit with "replit: " prefix.

## Purpose
Generates multiple images of the same AI character/face with consistent appearance across different scenes, outfits, and poses. Uses IP-Adapter + fal.ai for reference-based generation. Essential for brand mascots, consistent model shots, and adult content performers.

## Stack
- Python 3.11+, fal-client, Pillow, requests, anthropic, python-dotenv, tqdm

## Ethics
- Characters must be clearly AI-generated — no real person likeness cloning
- Adult characters must be explicitly adult (add age descriptors in prompts: "adult woman", "mature")
- NY AI Transparency Act (June 9, 2026): synthetic performer disclosure required in ads
- CSAM absolute ban: safety_check() runs on every prompt

## Tasks

### 1. safety_check.py
Same as other pipelines — blocks minor-related terms. REQUIRED before every generation.

### 2. character_creator.py
Define a reusable AI character:
```python
def create_character(
    description: str,       # "adult woman, 30s, dark hair, brown eyes, athletic"
    style: str,             # "photorealistic | anime | illustration"
    name: str,              # used as folder name and trigger word
    adult: bool = False,
) -> dict:
    """Generates 4 reference images of the character. Returns character profile dict."""
```
Saves to `characters/{name}/`: 4 reference images + `profile.json`.

### 3. scene_generator.py
Generate character in new scenes using IP-Adapter reference:
```python
def generate_scene(
    character_name: str,
    scene_prompt: str,      # "on a NYC rooftop at sunset, wearing casual outfit"
    reference_image: str = None,  # uses best reference from character profile if None
    adult: bool = False,
) -> str:
    """Returns path to generated image."""
```
Uses `fal-ai/ip-adapter-face-id` or `fal-ai/flux/dev` with reference conditioning.

### 4. consistency_checker.py
Score face consistency across generated images:
```python
def check_consistency(image_paths: list[str], reference_path: str) -> list[float]:
    """Returns similarity scores 0-1. Flags images below 0.6 threshold."""
```
Uses basic Pillow image comparison + histogram distance (no ML needed for MVP).

### 5. batch_scene_generator.py
Generate a full photo shoot for a character:
```python
SCENE_TEMPLATES = {
    "lifestyle": ["coffee shop", "gym", "city street", "rooftop", "beach"],
    "fashion": ["studio white bg", "urban brick wall", "luxury interior"],
    "adult": ["bedroom", "pool", "lingerie studio"],  # only if adult=True
}
def generate_shoot(character_name: str, template: str, adult=False) -> list[str]:
    """Generates full scene set. Returns list of output paths."""
```

### 6. character_gallery.py
Create an HTML gallery page showing all scenes for a character:
```python
def export_gallery(character_name: str) -> str:
    """Returns path to outputs/{character}/gallery.html"""
```

### 7. main.py CLI
```
python main.py create --name "aria" --desc "adult woman, 28, dark hair, confident" --style photorealistic
python main.py scene --character aria --scene "NYC rooftop at golden hour"
python main.py shoot --character aria --template lifestyle
python main.py shoot --character aria --template adult --adult
python main.py gallery --character aria
python main.py check --character aria  # consistency report
```

### 8. requirements.txt
```
fal-client>=0.5.0
anthropic>=0.39.0
Pillow>=10.4.0
requests>=2.31.0
python-dotenv>=1.0.0
tqdm>=4.66.0
```

### 9. .env.example
```
FAL_KEY=
ANTHROPIC_API_KEY=
```
