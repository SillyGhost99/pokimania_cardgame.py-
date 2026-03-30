╔══════════════════════════════════════════════════════════════════════╗
║                        P O K E M A N I A                           ║
║              A Pokémon Card-Battle Game (Tkinter GUI)               ║
╚══════════════════════════════════════════════════════════════════════╝

DESIGN OVERVIEW
===============

Ranking System
--------------
Each Pokémon has a global rank (1 = strongest overall, 50 = weakest).
Rank is derived from the sum of all stats (attack, defense, stamina,
speed, special).  Higher total stats → lower (better) rank number.
Some Pokémon are specialists: e.g., high attack but low defense. This
makes stat-choice strategic — you might beat a higher-ranked Pokémon
if you pick the right stat.

Battle Flow (4 rounds + optional 5th tiebreaker)
-------------------------------------------------
1. Both player and bot draw 10 random cards from the pool of 50.
2. Each match consists of 4 main rounds:
   - Both sides pick 1 card, then 1 stat to compare.
   - Higher stat wins the round.  Ties broken by small random factor.
3. If the score is 2-2 after 4 rounds, a 5th tiebreaker round occurs:
   - The player may reuse a previously played card OR pick from
     remaining unplayed cards.
   - The stat-chooser rotates: whoever chose the stat in round 4
     does NOT choose in round 5; the opponent chooses instead.
4. Winner of the 5th round wins the match.

Visual Effects
--------------
Pixel-art style sprites are drawn on a Tkinter canvas.  Each round
shows both Pokémon squaring off with animated "fight" effects and
special-attack flourishes.

Code Structure
--------------
Classes: Pokemon, Card, Deck, Player, Bot, BattleGUI
Entry point: main()
"""

import random
import math
import os
import tkinter as tk
from tkinter import messagebox

# Try to import PIL for image-based sprites
try:
    from PIL import Image, ImageTk
    HAS_PIL = True
except ImportError:
    HAS_PIL = False
    print("  [INFO] Pillow not installed — using canvas pixel art for sprites.")
    print("         Install with: pip install Pillow")

# ── Sprite image cache & loader ──
SCRIPT_DIR = os.path.dirname(os.path.abspath(__file__))
SPRITES_DIR = os.path.join(SCRIPT_DIR, "sprites")
GENERATED_DIR = os.path.join(
    os.path.expanduser("~"), ".gemini", "antigravity", "brain",
    "1a6f9e51-8d4c-4bb4-a028-fdf4076046b4"
)
_sprite_cache: dict[str, "ImageTk.PhotoImage"] = {}


def load_sprite_image(pokemon_name: str, size: int = 120):
    """Try to load a sprite image for the given Pokémon name.
    Checks sprites/ folder first, then the generated images folder.
    Returns an ImageTk.PhotoImage or None if not found / PIL unavailable.
    """
    if not HAS_PIL:
        return None

    key = f"{pokemon_name}_{size}"
    if key in _sprite_cache:
        return _sprite_cache[key]

    name_lower = pokemon_name.lower()

    # Search paths in priority order
    search_paths = [
        os.path.join(SPRITES_DIR, f"{name_lower}.png"),
        os.path.join(SPRITES_DIR, f"{name_lower}.jpg"),
    ]
    # Also check generated images folder (name_sprite_*.png pattern)
    if os.path.isdir(GENERATED_DIR):
        for f in os.listdir(GENERATED_DIR):
            if f.startswith(f"{name_lower}_sprite") and f.endswith(".png"):
                search_paths.append(os.path.join(GENERATED_DIR, f))

    for path in search_paths:
        if os.path.isfile(path):
            try:
                img = Image.open(path)
                img = img.resize((size, size), Image.LANCZOS)
                photo = ImageTk.PhotoImage(img)
                _sprite_cache[key] = photo
                return photo
            except Exception:
                continue

    return None

# ──────────────────────────────────────────────────────────────────────
#  POKÉMON DATA — 50 Pokémon (extensible pattern for 300-400)
# ──────────────────────────────────────────────────────────────────────

# Format: (name, type, attack, defense, stamina, speed, special)
RAW_POKEMON_DATA = [
    # ── Fire types ──
    ("Charizard",   "Fire",    84, 78, 78, 80, 109),
    ("Arcanine",    "Fire",    110, 80, 90, 95, 80),
    ("Typhlosion",  "Fire",    84, 78, 78, 80, 109),
    ("Blaziken",    "Fire",    120, 70, 80, 80, 110),
    ("Infernape",   "Fire",    104, 71, 76, 108, 104),
    ("Houndoom",    "Fire",    90, 50, 75, 95, 110),
    ("Magmortar",   "Fire",    95, 67, 75, 83, 125),
    ("Flareon",     "Fire",    130, 60, 65, 65, 95),
    ("Ninetales",   "Fire",    76, 75, 73, 100, 81),
    ("Rapidash",    "Fire",    100, 70, 65, 105, 80),

    # ── Water types ──
    ("Blastoise",   "Water",   83, 100, 79, 68, 85),
    ("Gyarados",    "Water",   125, 79, 95, 81, 60),
    ("Feraligatr",  "Water",   105, 100, 85, 78, 79),
    ("Swampert",    "Water",   110, 90, 100, 60, 85),
    ("Empoleon",    "Water",   86, 88, 84, 60, 111),
    ("Vaporeon",    "Water",   65, 60, 130, 65, 110),
    ("Lapras",      "Water",   85, 80, 130, 60, 85),
    ("Starmie",     "Water",   75, 85, 60, 115, 100),
    ("Kingdra",     "Water",   95, 95, 75, 85, 95),
    ("Milotic",     "Water",   60, 79, 95, 81, 100),

    # ── Grass types ──
    ("Venusaur",    "Grass",   82, 83, 80, 80, 100),
    ("Meganium",    "Grass",   82, 100, 80, 80, 83),
    ("Sceptile",    "Grass",   85, 65, 70, 120, 105),
    ("Torterra",    "Grass",   109, 105, 95, 56, 75),
    ("Roserade",    "Grass",   70, 65, 60, 90, 125),
    ("Leafeon",     "Grass",   110, 130, 65, 95, 60),
    ("Breloom",     "Grass",   130, 80, 60, 70, 60),
    ("Exeggutor",   "Grass",   95, 85, 95, 55, 125),
    ("Tangrowth",   "Grass",   100, 125, 100, 50, 110),
    ("Victreebel",  "Grass",   105, 65, 80, 70, 100),

    # ── Electric types ──
    ("Pikachu",     "Electric", 55, 40, 35, 90, 50),
    ("Raichu",      "Electric", 90, 55, 60, 110, 90),
    ("Jolteon",     "Electric", 65, 60, 65, 130, 110),
    ("Electivire",  "Electric", 123, 67, 75, 95, 95),
    ("Luxray",      "Electric", 120, 79, 80, 70, 95),
    ("Ampharos",    "Electric", 75, 85, 90, 55, 115),
    ("Manectric",   "Electric", 75, 60, 70, 105, 105),
    ("Magnezone",   "Electric", 70, 115, 70, 60, 130),

    # ── Psychic / Ghost / Dark types ──
    ("Alakazam",    "Psychic",  50, 45, 55, 120, 135),
    ("Gengar",      "Ghost",    65, 60, 60, 110, 130),
    ("Metagross",   "Psychic",  135, 130, 80, 70, 95),
    ("Gardevoir",   "Psychic",  65, 65, 68, 80, 125),
    ("Espeon",      "Psychic",  65, 60, 65, 110, 130),
    ("Umbreon",     "Dark",     65, 110, 95, 65, 60),
    ("Absol",       "Dark",     130, 60, 65, 75, 75),
    ("Dusknoir",    "Ghost",    100, 135, 45, 45, 65),

    # ── Fighting / Dragon / Normal types ──
    ("Dragonite",   "Dragon",   134, 95, 91, 80, 100),
    ("Salamence",   "Dragon",   135, 80, 95, 100, 110),
    ("Machamp",     "Fighting", 130, 80, 90, 55, 65),
    ("Snorlax",     "Normal",   110, 65, 160, 30, 65),
]


# ──────────────────────────────────────────────────────────────────────
#  CLASSES
# ──────────────────────────────────────────────────────────────────────

class Pokemon:
    """Represents a single Pokémon with its name, type, stats, and rank."""

    STAT_NAMES = ["attack", "defense", "stamina", "speed", "special"]

    def __init__(self, name: str, ptype: str, attack: int, defense: int,
                 stamina: int, speed: int, special: int, rank: int = 0):
        self.name = name
        self.ptype = ptype
        self.attack = attack
        self.defense = defense
        self.stamina = stamina
        self.speed = speed
        self.special = special
        self.rank = rank
        self.total = attack + defense + stamina + speed + special

    def get_stat(self, stat_name: str) -> int:
        """Return the value of the named stat."""
        return getattr(self, stat_name)

    def __repr__(self):
        return (f"{self.name} (Rank #{self.rank} | {self.ptype}) "
                f"ATK:{self.attack} DEF:{self.defense} STA:{self.stamina} "
                f"SPD:{self.speed} SPL:{self.special}")


class Card:
    """Wraps a Pokémon object for deck / hand management."""

    def __init__(self, pokemon: Pokemon):
        self.pokemon = pokemon
        self.used = False   # True once played in a round

    def mark_used(self):
        self.used = True

    def __repr__(self):
        status = " [USED]" if self.used else ""
        return f"[Card] {self.pokemon}{status}"


class Deck:
    """Pool of all Pokémon; can draw random hands."""

    def __init__(self, pokemon_list: list[Pokemon]):
        self.pool = list(pokemon_list)

    def draw_hand(self, count: int = 10) -> list[Card]:
        """Draw `count` random Pokémon from the pool and return as Cards."""
        chosen = random.sample(self.pool, min(count, len(self.pool)))
        return [Card(p) for p in chosen]


class Player:
    """Represents a human or bot player."""

    def __init__(self, name: str, is_bot: bool = False):
        self.name = name
        self.is_bot = is_bot
        self.hand: list[Card] = []
        self.wins = 0

    def available_cards(self) -> list[Card]:
        """Return cards not yet used."""
        return [c for c in self.hand if not c.used]

    def all_cards_for_tiebreaker(self) -> list[Card]:
        """For the 5th round, the player may reuse any card."""
        return list(self.hand)


# ──────────────────────────────────────────────────────────────────────
#  PIXEL ART SPRITES (drawn on Tkinter Canvas)
# ──────────────────────────────────────────────────────────────────────

# Type → color palette
TYPE_COLORS = {
    "Fire":     {"body": "#FF6B35", "accent": "#FFD166", "outline": "#C13A00",
                 "attack": "#FF4500", "bg": "#FFF0E5"},
    "Water":    {"body": "#4E94CE", "accent": "#A8DADC", "outline": "#1D3557",
                 "attack": "#00B4D8", "bg": "#E5F0FF"},
    "Grass":    {"body": "#6DBE45", "accent": "#A7E855", "outline": "#2D6A1E",
                 "attack": "#38B000", "bg": "#E5FFE5"},
    "Electric": {"body": "#FFD23F", "accent": "#FFF175", "outline": "#B89600",
                 "attack": "#FFE800", "bg": "#FFFFF0"},
    "Psychic":  {"body": "#F472B6", "accent": "#FBCFE8", "outline": "#9D174D",
                 "attack": "#E040FB", "bg": "#FFF0F5"},
    "Ghost":    {"body": "#7C3AED", "accent": "#C4B5FD", "outline": "#4C1D95",
                 "attack": "#8B5CF6", "bg": "#F0E5FF"},
    "Dark":     {"body": "#4A4A4A", "accent": "#888888", "outline": "#1A1A1A",
                 "attack": "#333333", "bg": "#F0F0F0"},
    "Dragon":   {"body": "#6366F1", "accent": "#A5B4FC", "outline": "#312E81",
                 "attack": "#4F46E5", "bg": "#E5E5FF"},
    "Fighting": {"body": "#DC2626", "accent": "#FCA5A5", "outline": "#7F1D1D",
                 "attack": "#EF4444", "bg": "#FFE5E5"},
    "Normal":   {"body": "#A3A3A3", "accent": "#D4D4D4", "outline": "#525252",
                 "attack": "#737373", "bg": "#F5F5F5"},
}


def draw_pixel_pokemon(canvas, cx, cy, pokemon: Pokemon, scale=3, facing="right"):
    """
    Draw a Pokémon sprite on the canvas.
    Tries to load an image file first (from sprites/ folder or generated images).
    Falls back to canvas pixel-art if no image is available.
    """
    # ── Try image-based sprite first ──
    sprite_size = max(80, scale * 25)
    sprite_img = load_sprite_image(pokemon.name, sprite_size)
    if sprite_img is not None:
        img_id = canvas.create_image(cx, cy, image=sprite_img, anchor="center")
        # Keep a reference to prevent garbage collection
        if not hasattr(canvas, '_sprite_refs'):
            canvas._sprite_refs = []
        canvas._sprite_refs.append(sprite_img)
        label = canvas.create_text(
            cx, cy + sprite_size // 2 + 12,
            text=pokemon.name,
            fill=TYPE_COLORS.get(pokemon.ptype, TYPE_COLORS["Normal"])["outline"],
            font=("Courier", 10, "bold")
        )
        return [img_id, label]

    # ── Fallback: canvas pixel art ──
    colors = TYPE_COLORS.get(pokemon.ptype, TYPE_COLORS["Normal"])
    s = scale
    flip = 1 if facing == "right" else -1

    items = []

    # ── Body (large block) ──
    body = canvas.create_rectangle(
        cx - 6*s, cy - 4*s, cx + 6*s, cy + 8*s,
        fill=colors["body"], outline=colors["outline"], width=2
    )
    items.append(body)

    # ── Head ──
    head = canvas.create_oval(
        cx - 5*s, cy - 12*s, cx + 5*s, cy - 2*s,
        fill=colors["body"], outline=colors["outline"], width=2
    )
    items.append(head)

    # ── Eyes ──
    eye_x = cx + flip * 2 * s
    eye = canvas.create_oval(
        eye_x - s, cy - 9*s, eye_x + s, cy - 7*s,
        fill="white", outline=colors["outline"]
    )
    items.append(eye)
    pupil = canvas.create_oval(
        eye_x - s//2, cy - 9*s + s//2, eye_x + s//2, cy - 7*s - s//2,
        fill=colors["outline"], outline=""
    )
    items.append(pupil)

    # ── Second eye ──
    eye_x2 = cx - flip * 1 * s
    eye2 = canvas.create_oval(
        eye_x2 - s, cy - 9*s, eye_x2 + s, cy - 7*s,
        fill="white", outline=colors["outline"]
    )
    items.append(eye2)

    # ── Accent marking (chest / belly) ──
    accent = canvas.create_rectangle(
        cx - 4*s, cy + 0*s, cx + 4*s, cy + 6*s,
        fill=colors["accent"], outline=""
    )
    items.append(accent)

    # ── Type-specific features ──
    if pokemon.ptype == "Fire":
        # Flame on tail
        for i in range(3):
            flame = canvas.create_polygon(
                cx - flip*6*s - flip*i*s, cy + 5*s,
                cx - flip*8*s - flip*i*s, cy - 2*s + i*2*s,
                cx - flip*5*s - flip*i*s, cy + 2*s,
                fill=["#FF4500", "#FFD166", "#FF6B35"][i], outline=""
            )
            items.append(flame)
    elif pokemon.ptype == "Water":
        # Fins
        fin = canvas.create_polygon(
            cx + 6*s, cy - 2*s,
            cx + 10*s, cy - 6*s,
            cx + 6*s, cy + 2*s,
            fill=colors["accent"], outline=colors["outline"]
        )
        items.append(fin)
    elif pokemon.ptype == "Grass":
        # Leaf on head
        leaf = canvas.create_polygon(
            cx, cy - 12*s,
            cx + 4*s, cy - 18*s,
            cx + 2*s, cy - 12*s,
            fill="#38B000", outline="#2D6A1E", width=1
        )
        items.append(leaf)
    elif pokemon.ptype == "Electric":
        # Lightning bolt
        bolt = canvas.create_polygon(
            cx - 2*s, cy - 14*s,
            cx + 1*s, cy - 10*s,
            cx - 1*s, cy - 10*s,
            cx + 2*s, cy - 6*s,
            cx, cy - 9*s,
            cx + 2*s, cy - 9*s,
            fill="#FFD23F", outline="#B89600", width=1
        )
        items.append(bolt)
    elif pokemon.ptype in ("Psychic", "Ghost"):
        # Aura glow
        for r in range(3, 0, -1):
            aura = canvas.create_oval(
                cx - (6+r*2)*s, cy - (12+r*2)*s,
                cx + (6+r*2)*s, cy + (8+r*2)*s,
                fill="", outline=colors["attack"],
                width=1, dash=(4, 4)
            )
            items.append(aura)
    elif pokemon.ptype == "Dragon":
        # Wings
        wing = canvas.create_polygon(
            cx - 6*s, cy - 4*s,
            cx - 14*s, cy - 12*s,
            cx - 10*s, cy - 2*s,
            fill=colors["accent"], outline=colors["outline"], width=1
        )
        items.append(wing)
        wing2 = canvas.create_polygon(
            cx + 6*s, cy - 4*s,
            cx + 14*s, cy - 12*s,
            cx + 10*s, cy - 2*s,
            fill=colors["accent"], outline=colors["outline"], width=1
        )
        items.append(wing2)
    elif pokemon.ptype == "Fighting":
        # Fists
        fist = canvas.create_oval(
            cx + flip*6*s, cy + 2*s, cx + flip*10*s, cy + 6*s,
            fill=colors["accent"], outline=colors["outline"], width=2
        )
        items.append(fist)

    # ── Arms ──
    arm_l = canvas.create_rectangle(
        cx - 8*s, cy + 0*s, cx - 6*s, cy + 6*s,
        fill=colors["body"], outline=colors["outline"], width=1
    )
    items.append(arm_l)
    arm_r = canvas.create_rectangle(
        cx + 6*s, cy + 0*s, cx + 8*s, cy + 6*s,
        fill=colors["body"], outline=colors["outline"], width=1
    )
    items.append(arm_r)

    # ── Legs ──
    leg_l = canvas.create_rectangle(
        cx - 4*s, cy + 8*s, cx - 1*s, cy + 13*s,
        fill=colors["body"], outline=colors["outline"], width=1
    )
    items.append(leg_l)
    leg_r = canvas.create_rectangle(
        cx + 1*s, cy + 8*s, cx + 4*s, cy + 13*s,
        fill=colors["body"], outline=colors["outline"], width=1
    )
    items.append(leg_r)

    # ── Name label ──
    label = canvas.create_text(
        cx, cy + 17*s,
        text=pokemon.name, fill=colors["outline"],
        font=("Courier", 10, "bold")
    )
    items.append(label)

    return items


# ──────────────────────────────────────────────────────────────────────
#  BATTLE GUI
# ──────────────────────────────────────────────────────────────────────

class BattleGUI:
    """Full game GUI — title screen, card selection, stat battles, animations."""

    # Layout constants
    WIDTH = 1100
    HEIGHT = 750
    CARD_W = 150
    CARD_H = 200

    # Color scheme
    BG = "#0F0F23"
    BG2 = "#1A1A3E"
    ACCENT = "#FFD700"
    TEXT = "#EAEAEA"
    WIN_COLOR = "#00E676"
    LOSE_COLOR = "#FF5252"
    TIE_COLOR = "#FFD740"
    CARD_BG = "#1E1E3F"
    CARD_HOVER = "#2A2A5F"
    CARD_SELECTED = "#FFD700"
    STAT_BTN = "#2C2C5A"
    STAT_HOVER = "#3E3E7E"

    def __init__(self, root: tk.Tk, deck: Deck):
        self.root = root
        self.deck = deck
        self.root.title("⚡ POKEMANIA ⚡")
        self.root.geometry(f"{self.WIDTH}x{self.HEIGHT}")
        self.root.configure(bg=self.BG)
        self.root.resizable(False, False)

        # ── Players ──
        self.player = Player("Player")
        self.bot = Player("Bot", is_bot=True)

        # ── Game state ──
        self.current_round = 0
        self.max_rounds = 4
        self.round_results = []          # list of ("player"/"bot"/"tie")
        self.selected_card_index = None
        self.player_card_this_round: Card | None = None
        self.bot_card_this_round: Card | None = None
        self.is_tiebreaker = False
        self.last_stat_chooser = "player"  # who chose stat last
        self.stat_chooser = "player"       # who chooses stat this round

        # ── Frames ──
        self.main_frame = tk.Frame(root, bg=self.BG)
        self.main_frame.pack(fill="both", expand=True)

        self._show_title_screen()

    # ══════════════════════════════════════════════════════════════════
    #  TITLE SCREEN
    # ══════════════════════════════════════════════════════════════════

    def _show_title_screen(self):
        self._clear_frame()

        canvas = tk.Canvas(self.main_frame, width=self.WIDTH, height=self.HEIGHT,
                           bg=self.BG, highlightthickness=0)
        canvas.pack()

        # Animated stars background
        for _ in range(80):
            x = random.randint(0, self.WIDTH)
            y = random.randint(0, self.HEIGHT)
            size = random.choice([1, 1, 1, 2, 2, 3])
            brightness = random.randint(100, 255)
            color = f"#{brightness:02x}{brightness:02x}{brightness:02x}"
            canvas.create_oval(x, y, x+size, y+size, fill=color, outline="")

        # Title text
        canvas.create_text(self.WIDTH//2, 160,
                           text="⚡ POKEMANIA ⚡",
                           font=("Courier", 52, "bold"),
                           fill=self.ACCENT)
        canvas.create_text(self.WIDTH//2, 220,
                           text="A Pokémon Card-Battle Game",
                           font=("Courier", 16),
                           fill="#AAAACC")

        # Rules summary
        rules = [
            "🃏  Draw 10 random Pokémon cards",
            "⚔️  Battle 4 rounds: pick a card & stat each round",
            "🏆  Higher stat wins the round",
            "🔥  Tied 2-2? A 5th tiebreaker round decides it all!",
        ]
        y_start = 300
        for i, rule in enumerate(rules):
            canvas.create_text(self.WIDTH//2, y_start + i * 35,
                               text=rule, font=("Courier", 13),
                               fill=self.TEXT)

        # Start button
        btn_x, btn_y = self.WIDTH//2, 520
        btn_w, btn_h = 200, 55
        canvas.create_rectangle(
            btn_x - btn_w//2, btn_y - btn_h//2,
            btn_x + btn_w//2, btn_y + btn_h//2,
            fill="#FFD700", outline="#FFA500", width=3
        )
        canvas.create_text(btn_x, btn_y,
                           text="START BATTLE",
                           font=("Courier", 18, "bold"),
                           fill="#1A1A3E")
        # Click region
        canvas.tag_bind("all", "<Button-1>", lambda e: self._check_start_click(e, btn_x, btn_y, btn_w, btn_h))
        canvas.bind("<Button-1>", lambda e: self._check_start_click(e, btn_x, btn_y, btn_w, btn_h))

    def _check_start_click(self, event, bx, by, bw, bh):
        if (bx - bw//2 <= event.x <= bx + bw//2 and
                by - bh//2 <= event.y <= by + bh//2):
            self._start_game()

    # ══════════════════════════════════════════════════════════════════
    #  GAME SETUP
    # ══════════════════════════════════════════════════════════════════

    def _start_game(self):
        """Initialize hands and start round 1."""
        self.player.hand = self.deck.draw_hand(10)
        self.bot.hand = self.deck.draw_hand(10)
        self.player.wins = 0
        self.bot.wins = 0
        self.current_round = 1
        self.round_results = []
        self.is_tiebreaker = False
        self.stat_chooser = "player"
        self._show_card_selection()

    # ══════════════════════════════════════════════════════════════════
    #  CARD SELECTION SCREEN
    # ══════════════════════════════════════════════════════════════════

    def _show_card_selection(self):
        self._clear_frame()
        self.selected_card_index = None

        # ── Header ──
        header = tk.Frame(self.main_frame, bg=self.BG2, height=60)
        header.pack(fill="x")
        header.pack_propagate(False)

        round_text = f"TIEBREAKER ROUND!" if self.is_tiebreaker else f"ROUND {self.current_round} of {self.max_rounds}"
        tk.Label(header, text=round_text,
                 font=("Courier", 20, "bold"), fg=self.ACCENT, bg=self.BG2
                 ).pack(side="left", padx=20, pady=10)

        score_text = f"Player {self.player.wins}  —  {self.bot.wins} Bot"
        tk.Label(header, text=score_text,
                 font=("Courier", 16), fg=self.TEXT, bg=self.BG2
                 ).pack(side="right", padx=20, pady=10)

        # ── Instruction ──
        tk.Label(self.main_frame,
                 text="Choose a Pokémon card to play this round:",
                 font=("Courier", 13), fg="#AAAACC", bg=self.BG
                 ).pack(pady=(15, 5))

        # ── Card area (scrollable-ish, 2 rows of 5) ──
        cards_frame = tk.Frame(self.main_frame, bg=self.BG)
        cards_frame.pack(pady=5, expand=True)

        if self.is_tiebreaker:
            available = self.player.all_cards_for_tiebreaker()
        else:
            available = self.player.available_cards()

        self._card_buttons = []
        for idx, card in enumerate(available):
            self._create_card_widget(cards_frame, card, idx, len(available))

        # ── Confirm button ──
        self.confirm_btn = tk.Button(
            self.main_frame, text="CONFIRM SELECTION",
            font=("Courier", 14, "bold"),
            fg=self.BG, bg="#555577", activebackground="#FFD700",
            relief="flat", bd=0, padx=20, pady=10,
            state="disabled",
            command=self._confirm_card_selection
        )
        self.confirm_btn.pack(pady=15)

    def _create_card_widget(self, parent, card: Card, idx: int, total: int):
        """Create a clickable card widget."""
        row = idx // 5
        col = idx % 5

        p = card.pokemon
        colors = TYPE_COLORS.get(p.ptype, TYPE_COLORS["Normal"])

        card_frame = tk.Frame(parent, bg=self.CARD_BG, bd=2,
                              relief="solid", highlightthickness=3,
                              highlightbackground="#333355",
                              highlightcolor=self.CARD_SELECTED)
        card_frame.grid(row=row, column=col, padx=6, pady=6)

        # ── Card content ──
        # Name
        tk.Label(card_frame, text=p.name, font=("Courier", 10, "bold"),
                 fg=colors["body"], bg=self.CARD_BG, width=14
                 ).pack(pady=(6, 0))

        # Type & Rank
        tk.Label(card_frame, text=f"{p.ptype} | Rank #{p.rank}",
                 font=("Courier", 8), fg="#999999", bg=self.CARD_BG
                 ).pack()

        # Separator
        tk.Frame(card_frame, bg=colors["body"], height=2).pack(fill="x", padx=8, pady=4)

        # Stats
        stats_frame = tk.Frame(card_frame, bg=self.CARD_BG)
        stats_frame.pack(padx=8, pady=2)

        stat_labels = [
            ("ATK", p.attack), ("DEF", p.defense), ("STA", p.stamina),
            ("SPD", p.speed), ("SPL", p.special),
        ]
        for sname, sval in stat_labels:
            sf = tk.Frame(stats_frame, bg=self.CARD_BG)
            sf.pack(fill="x")
            tk.Label(sf, text=f"{sname}:", font=("Courier", 8),
                     fg="#888888", bg=self.CARD_BG, anchor="w", width=5
                     ).pack(side="left")
            # Stat bar
            bar_canvas = tk.Canvas(sf, width=60, height=8, bg="#222244",
                                   highlightthickness=0)
            bar_canvas.pack(side="left", padx=(2, 4))
            bar_w = int(60 * min(sval / 160, 1.0))
            bar_canvas.create_rectangle(0, 0, bar_w, 8, fill=colors["body"], outline="")
            tk.Label(sf, text=str(sval), font=("Courier", 8),
                     fg=self.TEXT, bg=self.CARD_BG, width=4
                     ).pack(side="left")

        # Total
        tk.Label(card_frame, text=f"Total: {p.total}",
                 font=("Courier", 8, "bold"), fg=self.ACCENT, bg=self.CARD_BG
                 ).pack(pady=(4, 6))

        # Used indicator
        if card.used:
            tk.Label(card_frame, text="♻ REUSE", font=("Courier", 8),
                     fg="#FF9800", bg=self.CARD_BG).pack()

        # ── Click binding ──
        def on_click(event, i=idx, frame=card_frame):
            self._select_card(i, frame)

        card_frame.bind("<Button-1>", on_click)
        for child in card_frame.winfo_children():
            child.bind("<Button-1>", on_click)
            for grandchild in child.winfo_children():
                grandchild.bind("<Button-1>", on_click)
                for gchild in grandchild.winfo_children():
                    gchild.bind("<Button-1>", on_click)

        self._card_buttons.append(card_frame)

    def _select_card(self, idx, frame):
        """Highlight the selected card."""
        # Reset all borders
        for cb in self._card_buttons:
            cb.configure(highlightbackground="#333355")

        # Highlight selected
        frame.configure(highlightbackground=self.CARD_SELECTED)
        self.selected_card_index = idx

        # Enable confirm
        self.confirm_btn.configure(state="normal", bg=self.ACCENT)

    def _confirm_card_selection(self):
        """Lock in the player's card, let bot pick, then go to stat selection."""
        if self.selected_card_index is None:
            return

        if self.is_tiebreaker:
            available = self.player.all_cards_for_tiebreaker()
        else:
            available = self.player.available_cards()

        self.player_card_this_round = available[self.selected_card_index]

        # Bot picks its card (simple AI: pick the strongest available card)
        if self.is_tiebreaker:
            bot_available = self.bot.all_cards_for_tiebreaker()
        else:
            bot_available = self.bot.available_cards()

        # Bot strategy: pick the card with the highest total stats
        bot_available_sorted = sorted(bot_available,
                                      key=lambda c: c.pokemon.total, reverse=True)
        self.bot_card_this_round = bot_available_sorted[0]

        # Mark cards as used (only if not already used from tiebreaker reuse)
        if not self.player_card_this_round.used:
            self.player_card_this_round.mark_used()
        if not self.bot_card_this_round.used:
            self.bot_card_this_round.mark_used()

        # Go to stat selection
        self._show_stat_selection()

    # ══════════════════════════════════════════════════════════════════
    #  STAT SELECTION SCREEN
    # ══════════════════════════════════════════════════════════════════

    def _show_stat_selection(self):
        self._clear_frame()

        p_poke = self.player_card_this_round.pokemon
        b_poke = self.bot_card_this_round.pokemon

        # ── Header ──
        header = tk.Frame(self.main_frame, bg=self.BG2, height=50)
        header.pack(fill="x")
        header.pack_propagate(False)

        round_text = "TIEBREAKER!" if self.is_tiebreaker else f"ROUND {self.current_round}"
        tk.Label(header, text=f"{round_text}  |  Choose a stat to compare!",
                 font=("Courier", 16, "bold"), fg=self.ACCENT, bg=self.BG2
                 ).pack(pady=10)

        # ── Battle arena canvas ──
        arena = tk.Canvas(self.main_frame, width=self.WIDTH, height=280,
                          bg=self.BG, highlightthickness=0)
        arena.pack()

        # Draw VS divider
        arena.create_text(self.WIDTH//2, 140, text="VS",
                          font=("Courier", 36, "bold"), fill="#FF5555")

        # Draw both Pokémon sprites
        draw_pixel_pokemon(arena, 200, 140, p_poke, scale=4, facing="right")
        draw_pixel_pokemon(arena, self.WIDTH - 200, 140, b_poke, scale=4, facing="left")

        # Player card info
        arena.create_text(200, 260, text=f"{p_poke.name}",
                          font=("Courier", 14, "bold"),
                          fill=TYPE_COLORS.get(p_poke.ptype, TYPE_COLORS["Normal"])["body"])
        arena.create_text(200, 278, text=f"Rank #{p_poke.rank} | {p_poke.ptype}",
                          font=("Courier", 10), fill="#999999")

        # Bot card info
        arena.create_text(self.WIDTH - 200, 260, text=f"{b_poke.name}",
                          font=("Courier", 14, "bold"),
                          fill=TYPE_COLORS.get(b_poke.ptype, TYPE_COLORS["Normal"])["body"])
        arena.create_text(self.WIDTH - 200, 278, text=f"Rank #{b_poke.rank} | {b_poke.ptype}",
                          font=("Courier", 10), fill="#999999")

        # Who chooses?
        if self.stat_chooser == "player":
            chooser_text = "🎯 YOUR TURN to pick a stat!"
        else:
            chooser_text = "🤖 BOT is choosing the stat..."

        tk.Label(self.main_frame, text=chooser_text,
                 font=("Courier", 14, "bold"), fg=self.ACCENT, bg=self.BG
                 ).pack(pady=(10, 5))

        if self.stat_chooser == "bot":
            # Bot picks the stat automatically
            self.root.after(1200, self._bot_picks_stat)
        else:
            # Player picks
            self._show_stat_buttons(p_poke, b_poke)

    def _show_stat_buttons(self, p_poke, b_poke):
        """Show buttons for the 5 stats."""
        btn_frame = tk.Frame(self.main_frame, bg=self.BG)
        btn_frame.pack(pady=10)

        stats = Pokemon.STAT_NAMES
        stat_display = {"attack": "⚔️ ATTACK", "defense": "🛡️ DEFENSE",
                        "stamina": "💚 STAMINA", "speed": "💨 SPEED",
                        "special": "✨ SPECIAL"}

        for i, stat in enumerate(stats):
            p_val = p_poke.get_stat(stat)
            b_val = b_poke.get_stat(stat)

            btn = tk.Button(
                btn_frame,
                text=f"{stat_display[stat]}\nYou: {p_val}  |  ???",
                font=("Courier", 11, "bold"),
                fg=self.TEXT, bg=self.STAT_BTN,
                activeforeground=self.ACCENT,
                activebackground=self.STAT_HOVER,
                relief="flat", bd=0,
                width=20, height=3,
                command=lambda s=stat: self._resolve_round(s)
            )
            btn.grid(row=0, column=i, padx=6, pady=4)

    def _bot_picks_stat(self):
        """Bot AI stat selection: pick the stat where it has the biggest advantage."""
        p_poke = self.player_card_this_round.pokemon
        b_poke = self.bot_card_this_round.pokemon

        best_stat = max(Pokemon.STAT_NAMES,
                        key=lambda s: b_poke.get_stat(s) - p_poke.get_stat(s))
        self._resolve_round(best_stat)

    # ══════════════════════════════════════════════════════════════════
    #  ROUND RESOLUTION + ANIMATION
    # ══════════════════════════════════════════════════════════════════

    def _resolve_round(self, chosen_stat: str):
        """Compare the chosen stat, determine winner, animate."""
        p_poke = self.player_card_this_round.pokemon
        b_poke = self.bot_card_this_round.pokemon
        p_val = p_poke.get_stat(chosen_stat)
        b_val = b_poke.get_stat(chosen_stat)

        # Determine winner (with tiebreaker)
        if p_val > b_val:
            winner = "player"
            self.player.wins += 1
        elif b_val > p_val:
            winner = "bot"
            self.bot.wins += 1
        else:
            # Random tiebreaker
            if random.random() < 0.5:
                winner = "player"
                self.player.wins += 1
            else:
                winner = "bot"
                self.bot.wins += 1

        self.round_results.append(winner)

        # Track who chose stat (for tiebreaker rule)
        self.last_stat_chooser = self.stat_chooser

        # Show battle animation
        self._show_battle_animation(chosen_stat, p_val, b_val, winner)

    def _show_battle_animation(self, stat_name: str, p_val: int, b_val: int, winner: str):
        """Animated battle result screen."""
        self._clear_frame()

        p_poke = self.player_card_this_round.pokemon
        b_poke = self.bot_card_this_round.pokemon

        canvas = tk.Canvas(self.main_frame, width=self.WIDTH, height=self.HEIGHT,
                           bg=self.BG, highlightthickness=0)
        canvas.pack()

        # Round header
        round_text = "⚡ TIEBREAKER ⚡" if self.is_tiebreaker else f"⚡ ROUND {self.current_round} ⚡"
        canvas.create_text(self.WIDTH//2, 35, text=round_text,
                           font=("Courier", 22, "bold"), fill=self.ACCENT)

        # Draw Pokémon sprites
        p_items = draw_pixel_pokemon(canvas, 220, 200, p_poke, scale=5, facing="right")
        b_items = draw_pixel_pokemon(canvas, self.WIDTH - 220, 200, b_poke, scale=5, facing="left")

        # VS
        canvas.create_text(self.WIDTH//2, 180, text="VS",
                           font=("Courier", 40, "bold"), fill="#FF3333")

        # Stat comparison
        stat_display = {"attack": "⚔️ ATTACK", "defense": "🛡️ DEFENSE",
                        "stamina": "💚 STAMINA", "speed": "💨 SPEED",
                        "special": "✨ SPECIAL"}
        canvas.create_text(self.WIDTH//2, 320,
                           text=f"Comparing: {stat_display[stat_name]}",
                           font=("Courier", 16, "bold"), fill="#CCCCEE")

        # Score bars
        bar_y = 370
        max_val = max(p_val, b_val, 1)

        # Player bar
        p_bar_w = int(300 * p_val / max_val)
        canvas.create_rectangle(self.WIDTH//2 - 20 - 300, bar_y,
                                self.WIDTH//2 - 20, bar_y + 30,
                                fill="#222244", outline="#444466")
        p_color = self.WIN_COLOR if winner == "player" else self.LOSE_COLOR
        canvas.create_rectangle(self.WIDTH//2 - 20 - p_bar_w, bar_y,
                                self.WIDTH//2 - 20, bar_y + 30,
                                fill=p_color, outline="")
        canvas.create_text(self.WIDTH//2 - 20 - 150, bar_y + 15,
                           text=f"{p_poke.name}: {p_val}",
                           font=("Courier", 12, "bold"), fill="white")

        # Bot bar
        b_bar_w = int(300 * b_val / max_val)
        canvas.create_rectangle(self.WIDTH//2 + 20, bar_y,
                                self.WIDTH//2 + 20 + 300, bar_y + 30,
                                fill="#222244", outline="#444466")
        b_color = self.WIN_COLOR if winner == "bot" else self.LOSE_COLOR
        canvas.create_rectangle(self.WIDTH//2 + 20, bar_y,
                                self.WIDTH//2 + 20 + b_bar_w, bar_y + 30,
                                fill=b_color, outline="")
        canvas.create_text(self.WIDTH//2 + 20 + 150, bar_y + 15,
                           text=f"{b_poke.name}: {b_val}",
                           font=("Courier", 12, "bold"), fill="white")

        # Winner announcement
        if winner == "player":
            result_text = f"🏆 {p_poke.name} WINS THIS ROUND!"
            result_color = self.WIN_COLOR
        else:
            result_text = f"💀 {b_poke.name} WINS THIS ROUND!"
            result_color = self.LOSE_COLOR

        canvas.create_text(self.WIDTH//2, 440,
                           text=result_text,
                           font=("Courier", 20, "bold"), fill=result_color)

        # Overall score
        canvas.create_text(self.WIDTH//2, 490,
                           text=f"Score:  Player {self.player.wins}  —  {self.bot.wins}  Bot",
                           font=("Courier", 16), fill=self.TEXT)

        # Round history
        history_y = 530
        canvas.create_text(self.WIDTH//2, history_y,
                           text="Round History:", font=("Courier", 12),
                           fill="#888888")
        for i, res in enumerate(self.round_results):
            marker = "🟢" if res == "player" else "🔴"
            rnd_label = "TB" if (i == 4) else str(i + 1)
            canvas.create_text(
                self.WIDTH//2 - 100 + i * 50, history_y + 25,
                text=f"R{rnd_label} {marker}",
                font=("Courier", 11), fill=self.TEXT
            )

        # ── Special attack visual flourish ──
        self._animate_special_attack(canvas, p_poke, b_poke, winner)

        # ── Continue button ──
        continue_btn_id = canvas.create_rectangle(
            self.WIDTH//2 - 120, 620, self.WIDTH//2 + 120, 665,
            fill=self.ACCENT, outline="#FFA500", width=2
        )
        continue_text_id = canvas.create_text(
            self.WIDTH//2, 643, text="CONTINUE ▶",
            font=("Courier", 16, "bold"), fill=self.BG
        )
        canvas.tag_bind(continue_btn_id, "<Button-1>", lambda e: self._next_round())
        canvas.tag_bind(continue_text_id, "<Button-1>", lambda e: self._next_round())

    def _animate_special_attack(self, canvas, p_poke, b_poke, winner):
        """Draw special attack particle effects."""
        # Winner's Pokémon gets a flashy burst
        win_poke = p_poke if winner == "player" else b_poke
        cx = 220 if winner == "player" else self.WIDTH - 220
        colors = TYPE_COLORS.get(win_poke.ptype, TYPE_COLORS["Normal"])

        # Particle burst
        for i in range(12):
            angle = i * (360 / 12)
            rad = math.radians(angle)
            for r in range(3):
                dist = 60 + r * 25
                px = cx + dist * math.cos(rad)
                py = 200 + dist * math.sin(rad)
                size = 6 - r * 2
                canvas.create_oval(
                    px - size, py - size, px + size, py + size,
                    fill=colors["attack"], outline=""
                )

        # Beam / line toward opponent
        target_x = self.WIDTH - 220 if winner == "player" else 220
        for i in range(5):
            offset = (i - 2) * 4
            canvas.create_line(
                cx + (30 if winner == "player" else -30), 200 + offset,
                target_x - (30 if winner == "player" else -30), 200 + offset,
                fill=colors["attack"], width=2, dash=(8, 4)
            )

    # ══════════════════════════════════════════════════════════════════
    #  ROUND FLOW CONTROL
    # ══════════════════════════════════════════════════════════════════

    def _next_round(self):
        """Determine what happens after a round ends."""
        if self.is_tiebreaker:
            # Tiebreaker just ended — show final result
            self._show_final_result()
            return

        if self.current_round >= self.max_rounds:
            # Check if tiebreaker is needed
            if self.player.wins == self.bot.wins:
                self.is_tiebreaker = True
                # Stat chooser switches from whoever chose in round 4
                self.stat_chooser = "bot" if self.last_stat_chooser == "player" else "player"
                self._show_card_selection()
            else:
                self._show_final_result()
        else:
            self.current_round += 1
            # Alternate stat chooser each round
            self.stat_chooser = "bot" if self.stat_chooser == "player" else "player"
            self._show_card_selection()

    # ══════════════════════════════════════════════════════════════════
    #  FINAL RESULT SCREEN
    # ══════════════════════════════════════════════════════════════════

    def _show_final_result(self):
        self._clear_frame()

        canvas = tk.Canvas(self.main_frame, width=self.WIDTH, height=self.HEIGHT,
                           bg=self.BG, highlightthickness=0)
        canvas.pack()

        # Background particles
        for _ in range(100):
            x = random.randint(0, self.WIDTH)
            y = random.randint(0, self.HEIGHT)
            size = random.choice([1, 2, 2, 3])
            brightness = random.randint(60, 200)
            color = f"#{brightness:02x}{brightness:02x}{brightness:02x}"
            canvas.create_oval(x, y, x+size, y+size, fill=color, outline="")

        if self.player.wins > self.bot.wins:
            title = "🏆 VICTORY! 🏆"
            subtitle = "You are the Pokémon Master!"
            title_color = self.WIN_COLOR
            # Gold confetti burst
            for _ in range(60):
                x = random.randint(100, self.WIDTH - 100)
                y = random.randint(100, 400)
                size = random.randint(3, 8)
                color = random.choice(["#FFD700", "#FFA500", "#FFE800",
                                       "#FF6B35", "#00E676"])
                canvas.create_rectangle(x, y, x+size, y+size,
                                        fill=color, outline="")
        else:
            title = "💀 DEFEAT 💀"
            subtitle = "The Bot wins this time..."
            title_color = self.LOSE_COLOR

        canvas.create_text(self.WIDTH//2, 150, text=title,
                           font=("Courier", 48, "bold"), fill=title_color)
        canvas.create_text(self.WIDTH//2, 210, text=subtitle,
                           font=("Courier", 18), fill=self.TEXT)

        # Final score
        canvas.create_text(self.WIDTH//2, 280,
                           text=f"FINAL SCORE",
                           font=("Courier", 14), fill="#888888")
        canvas.create_text(self.WIDTH//2, 320,
                           text=f"Player  {self.player.wins}  :  {self.bot.wins}  Bot",
                           font=("Courier", 28, "bold"), fill=self.ACCENT)

        # Round-by-round recap
        recap_y = 390
        canvas.create_text(self.WIDTH//2, recap_y,
                           text="─── Round Recap ───",
                           font=("Courier", 14), fill="#666688")
        for i, res in enumerate(self.round_results):
            rnd_label = "Tiebreaker" if i == 4 else f"Round {i+1}"
            if res == "player":
                icon, color = "🟢 Player Won", self.WIN_COLOR
            else:
                icon, color = "🔴 Bot Won", self.LOSE_COLOR
            canvas.create_text(self.WIDTH//2, recap_y + 30 + i * 28,
                               text=f"{rnd_label}: {icon}",
                               font=("Courier", 13), fill=color)

        # Play again & Quit buttons
        btn_y = 600

        # Play Again
        pa_btn = canvas.create_rectangle(
            self.WIDTH//2 - 220, btn_y, self.WIDTH//2 - 20, btn_y + 50,
            fill=self.ACCENT, outline="#FFA500", width=2
        )
        pa_text = canvas.create_text(
            self.WIDTH//2 - 120, btn_y + 25,
            text="PLAY AGAIN",
            font=("Courier", 16, "bold"), fill=self.BG
        )
        canvas.tag_bind(pa_btn, "<Button-1>", lambda e: self._start_game())
        canvas.tag_bind(pa_text, "<Button-1>", lambda e: self._start_game())

        # Quit
        q_btn = canvas.create_rectangle(
            self.WIDTH//2 + 20, btn_y, self.WIDTH//2 + 220, btn_y + 50,
            fill="#FF5252", outline="#CC0000", width=2
        )
        q_text = canvas.create_text(
            self.WIDTH//2 + 120, btn_y + 25,
            text="QUIT",
            font=("Courier", 16, "bold"), fill="white"
        )
        canvas.tag_bind(q_btn, "<Button-1>", lambda e: self.root.quit())
        canvas.tag_bind(q_text, "<Button-1>", lambda e: self.root.quit())

    # ══════════════════════════════════════════════════════════════════
    #  UTILITIES
    # ══════════════════════════════════════════════════════════════════

    def _clear_frame(self):
        """Remove all children from the main frame."""
        for widget in self.main_frame.winfo_children():
            widget.destroy()


# ──────────────────────────────────────────────────────────────────────
#  POKÉMON POOL INITIALIZATION
# ──────────────────────────────────────────────────────────────────────

def build_pokemon_pool() -> list[Pokemon]:
    """
    Build the Pokémon pool from raw data, compute ranks based on
    total stats (higher total → better/lower rank number).
    """
    pokemon_list = []
    for name, ptype, atk, dfn, sta, spd, spl in RAW_POKEMON_DATA:
        pokemon_list.append(Pokemon(name, ptype, atk, dfn, sta, spd, spl))

    # Sort by total descending → assign rank (1 = strongest)
    pokemon_list.sort(key=lambda p: p.total, reverse=True)
    for i, poke in enumerate(pokemon_list):
        poke.rank = i + 1

    return pokemon_list


# ──────────────────────────────────────────────────────────────────────
#  MAIN
# ──────────────────────────────────────────────────────────────────────

def main():
    """
    Entry point for Pokemania.

    1. Builds the Pokémon pool (50 Pokémon with stats and ranks).
    2. Creates the Deck.
    3. Launches the Tkinter GUI for the card-battle game.
    """
    # Build pool and deck
    pool = build_pokemon_pool()
    deck = Deck(pool)

    # Print pool info to console
    print("=" * 60)
    print("  ⚡ POKEMANIA — Pokémon Card-Battle Game ⚡")
    print("=" * 60)
    print(f"\n  Loaded {len(pool)} Pokémon into the battle pool.\n")
    print("  Top 10 Pokémon by Rank:")
    print("  " + "-" * 56)
    for p in pool[:10]:
        print(f"  #{p.rank:>2}  {p.name:<12} {p.ptype:<10} "
              f"ATK:{p.attack:>3} DEF:{p.defense:>3} "
              f"STA:{p.stamina:>3} SPD:{p.speed:>3} SPL:{p.special:>3} "
              f"TOT:{p.total:>3}")
    print("  " + "-" * 56)
    print("\n  Launching GUI...\n")

    # Launch GUI
    root = tk.Tk()
    app = BattleGUI(root, deck)
    root.mainloop()


if __name__ == "__main__":
    main()
