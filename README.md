# 🔴 Pokémon Quiz

**Who's that Pokémon?** A fast-paced quiz game to test your Pokémon Trainer knowledge!

🎮 **[Play Live](https://alakshit-pradhan.github.io/pokemon-quiz/)**

## Features

- 🌱 **3 difficulty modes:** Easy (Kanto classics), Medium (Gen 2-4 favourites), Hard (deep-cut Pokémon from every region)
- 📸 **50 questions per round** — random Pokémon from the chosen difficulty pool
- ⏱️ **8-second timer** per question — watch the bar turn amber, then red
- 🎯 **4 multiple-choice options** with shuffled decoys from the same difficulty
- 🏆 **Grading system:** S / A / B / C / D based on your score
- ⌨️ **Keyboard shortcuts:** Press 1–4 to select an option (desktop)
- 📱 **Fully mobile-responsive** — thumb-friendly tap targets

## Tech Stack

- **Pure HTML / CSS / JavaScript** — no frameworks, no build step
- **Pokémon sprites** from the [PokeAPI](https://pokeapi.co/) CDN
- **Google Fonts** — Fredoka, Press Start 2P, Rubik
- Single self-contained `index.html` file

## How It Works

Each Pokémon in the 200+ entry database is hand-tagged by difficulty. When you start a round, 50 are picked at random from your chosen tier and shuffled. For each question, 3 random decoy names from the same tier join the correct name as options. Images preload in the background for smooth transitions.

## Run Locally

No build required — just open `index.html` in any modern browser.

```bash
git clone https://github.com/Alakshit-Pradhan/pokemon-quiz.git
cd pokemon-quiz
# Open index.html in your browser, or:
python3 -m http.server 8000  # then visit http://localhost:8000
```

## Credits

- **Design & code:** [Alakshit Pradhan](https://github.com/Alakshit-Pradhan)
- **Pokémon artwork:** [PokéAPI Sprites](https://github.com/PokeAPI/sprites)
- **Fonts:** Google Fonts

> Pokémon and all related names are trademarks of Nintendo / Game Freak / The Pokémon Company. This is a non-commercial fan project.
