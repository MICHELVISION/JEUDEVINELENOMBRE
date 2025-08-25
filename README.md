import random
import requests
import pyttsx3
from deep_translator import GoogleTranslator
import tkinter as tk
from tkinter import scrolledtext, simpledialog, Toplevel, ttk
import json
import os

# ==============================
#  Configuration voix du robot
# ==============================
moteur_voix = pyttsx3.init()
moteur_voix.setProperty("rate", 170)
moteur_voix.setProperty("volume", 1)

for v in moteur_voix.getProperty("voices"):
    if "fr" in v.id.lower():
        moteur_voix.setProperty("voice", v.id)
        break

def parler(texte):
    zone_chat.insert(tk.END, f"🤖 {texte}\n", "robot")
    zone_chat.see(tk.END)
    moteur_voix.say(texte)
    moteur_voix.runAndWait()

# ==============================
#  Blagues et traduction
# ==============================
blagues_locales = [
    "Pourquoi les plongeurs plongent-ils toujours en arrière ? Parce que sinon ils tombent dans le bateau 😂",
    "Quelle est la femelle du hamster ? → L’Amsterdam 🤣",
    "Pourquoi les maths adorent TikTok ? Parce qu’il y a trop de ‘functions’ !",
    "J’ai voulu raconter une blague sur le temps… mais elle n’était pas au point ⏳😅",
    "Tu connais la blague du lit ? Non ? Tant mieux, elle est trop courte 😴"
]

def traduire_vers_fr(texte):
    try:
        return GoogleTranslator(source="auto", target="fr").translate(texte)
    except:
        return texte

def raconter_blagues():
    try:
        url = "https://v2.jokeapi.dev/joke/Any?type=single"
        data = requests.get(url, timeout=5).json()
        if "joke" in data:
            blague = traduire_vers_fr(data["joke"])
            parler(blague)
            return
    except:
        pass
    parler(random.choice(blagues_locales))

# ==============================
#  Chatbot
# ==============================
def chatbot(message):
    reponses = {
        "salut": "Yo 👋 Comment ça va ?",
        "ça va": "Toujours au top, surtout avec toi 😎🔥",
        "qui es tu": "Je suis ton robot préféré, mélange entre TikTok et Einstein 🤖",
        "merci": "De rien poto, je facture juste 1 sourire 😁",
        "bye": "Ciao ciao 👋 À la prochaine !"
    }
    texte = reponses.get(message.lower(), "Haha, je comprends pas trop… mais j’approuve 🤪")
    parler(texte)

# ==============================
#  Gestion du classement (JSON)
# ==============================
FICHIER_SCORES = "scores.json"

def charger_scores():
    if os.path.exists(FICHIER_SCORES):
        with open(FICHIER_SCORES, "r", encoding="utf-8") as f:
            return json.load(f)
    return {}

def sauvegarder_scores():
    with open(FICHIER_SCORES, "w", encoding="utf-8") as f:
        json.dump(classement, f, indent=4, ensure_ascii=False)

classement = charger_scores()

# ==============================
#  Jeu multi-joueurs
# ==============================
nombre_secret = random.randint(1, 50)
essais = 0
pseudo = None

def demander_pseudo():
    global pseudo
    pseudo = simpledialog.askstring("Pseudo", "Entre ton pseudo :")
    if not pseudo:
        pseudo = "Anonyme"
    parler(f"Bienvenue {pseudo} ! 🎉 Essaie de battre ton record 💪")

def trier_classement():
    return sorted(classement.items(), key=lambda x: x[1])

def afficher_classement():
    if not classement:
        return "Aucun score pour le moment."
    tri = trier_classement()
    texte = "🏆 Classement :\n"
    for i, (joueur, score) in enumerate(tri, 1):
        texte += f"{i}. {joueur} → {score} essais\n"
    return texte

def envoyer_message():
    global essais, nombre_secret, pseudo
    user_input = entree.get()
    entree.delete(0, tk.END)

    if not user_input.strip():
        return

    zone_chat.insert(tk.END, f"👤 {pseudo}: {user_input}\n", "user")
    zone_chat.see(tk.END)

    if user_input.isdigit():
        choix = int(user_input)
        essais += 1

        if choix < nombre_secret:
            parler("😏 Trop petit ! Essaie plus grand.")
            raconter_blagues()
        elif choix > nombre_secret:
            parler("😜 Trop grand ! Essaie plus petit.")
            raconter_blagues()
        else:
            parler(f"🎉 BRAVO {pseudo} ! Tu as trouvé le nombre {nombre_secret} en {essais} essais 🎉")

            if pseudo not in classement or essais < classement[pseudo]:
                classement[pseudo] = essais
                sauvegarder_scores()
                parler("🔥 Nouveau record personnel enregistré !")
            else:
                parler(f"Ton record reste {classement[pseudo]} essais.")

            parler(afficher_classement())

            nombre_secret = random.randint(1, 50)
            essais = 0
    else:
        chatbot(user_input)

# ==============================
#  Fenêtre classement (couleurs)
# ==============================
def ouvrir_classement():
    top = Toplevel(fenetre)
    top.title("🏆 Classement")
    top.configure(bg="#111")

    colonnes = ("Rang", "Joueur", "Score (essais)")
    tableau = ttk.Treeview(top, columns=colonnes, show="headings", height=10)

    tableau.heading("Rang", text="🏅 Rang")
    tableau.heading("Joueur", text="👤 Joueur")
    tableau.heading("Score (essais)", text="🎯 Score")

    tableau.column("Rang", width=100, anchor="center")
    tableau.column("Joueur", width=150, anchor="center")
    tableau.column("Score (essais)", width=120, anchor="center")

    style = ttk.Style()
    style.configure("Treeview", rowheight=30, font=("Arial", 12))
    style.map("Treeview")

    tableau.tag_configure("gold", background="#FFD700", foreground="black")
    tableau.tag_configure("silver", background="#C0C0C0", foreground="black")
    tableau.tag_configure("bronze", background="#CD7F32", foreground="black")

    medailles = {1: ("🥇", "gold"), 2: ("🥈", "silver"), 3: ("🥉", "bronze")}
    for i, (joueur, score) in enumerate(trier_classement(), 1):
        if i in medailles:
            icone, couleur = medailles[i]
            tableau.insert("", "end", values=(f"{icone} {i}", joueur, score), tags=(couleur,))
        else:
            tableau.insert("", "end", values=(str(i), joueur, score))

    tableau.pack(padx=20, pady=20)

# ==============================
#  Interface Tkinter
# ==============================
fenetre = tk.Tk()
fenetre.title("🎮 Devine le Nombre - Robot TikTok 😎🔥")
fenetre.configure(bg="#0f0f0f")

zone_chat = scrolledtext.ScrolledText(
    fenetre, wrap=tk.WORD, width=60, height=20,
    font=("Comic Sans MS", 12), bg="#1c1c1c", fg="white"
)
zone_chat.tag_config("robot", foreground="#00ffcc", font=("Comic Sans MS", 12, "bold"))
zone_chat.tag_config("user", foreground="#ff4d94", font=("Comic Sans MS", 12, "bold"))
zone_chat.pack(padx=10, pady=10)

cadre = tk.Frame(fenetre, bg="#0f0f0f")
cadre.pack(pady=10)

entree = tk.Entry(cadre, width=40, font=("Arial", 14), bg="#262626", fg="white", insertbackground="white")
entree.pack(side=tk.LEFT, padx=10)

bouton_envoyer = tk.Button(
    cadre, text="Envoyer 🚀", command=envoyer_message,
    font=("Arial", 12, "bold"), bg="#ff0050", fg="white", activebackground="#ff3385"
)
bouton_envoyer.pack(side=tk.LEFT)

bouton_classement = tk.Button(
    cadre, text="Voir Classement 🏆", command=ouvrir_classement,
    font=("Arial", 12, "bold"), bg="#00cc66", fg="white", activebackground="#00e68a"
)
bouton_classement.pack(side=tk.LEFT, padx=10)

demander_pseudo()
parler("🎉 Bienvenue dans le jeu DEVINE LE NOMBRE 🎉 Je pense à un nombre entre 1 et 50. Essaie de le trouver ! 💡 Tu peux aussi me parler comme à un chatbot 😉")

fenetre.mainloop()
def animer_top3(tableau, item, couleur1, couleur2, nb=6):
    """Fait clignoter une ligne du tableau TOP 3"""
    def toggle(n):
        if n > 0:
            actuel = tableau.item(item, "tags")
            if "blink1" in actuel:
                tableau.item(item, tags=("blink2",))
            else:
                tableau.item(item, tags=("blink1",))
            tableau.after(300, toggle, n-1)
    tableau.tag_configure("blink1", background=couleur1, foreground="black")
    tableau.tag_configure("blink2", background=couleur2, foreground="white")
    toggle(nb)
def ouvrir_classement():
    top = Toplevel(fenetre)
    top.title("🏆 Classement")
    top.configure(bg="#111")

    colonnes = ("Rang", "Joueur", "Score (essais)")
    tableau = ttk.Treeview(top, columns=colonnes, show="headings", height=10)

    tableau.heading("Rang", text="🏅 Rang")
    tableau.heading("Joueur", text="👤 Joueur")
    tableau.heading("Score (essais)", text="🎯 Score")

    tableau.column("Rang", width=100, anchor="center")
    tableau.column("Joueur", width=150, anchor="center")
    tableau.column("Score (essais)", width=120, anchor="center")

    style = ttk.Style()
    style.configure("Treeview", rowheight=30, font=("Arial", 12))

    tableau.tag_configure("gold", background="#FFD700", foreground="black")
    tableau.tag_configure("silver", background="#C0C0C0", foreground="black")
    tableau.tag_configure("bronze", background="#CD7F32", foreground="black")

    medailles = {1: ("🥇", "gold"), 2: ("🥈", "silver"), 3: ("🥉", "bronze")}
    item_pseudo = None  

    for i, (joueur, score) in enumerate(trier_classement(), 1):
        if i in medailles:
            icone, couleur = medailles[i]
            item = tableau.insert("", "end", values=(f"{icone} {i}", joueur, score), tags=(couleur,))
            if joueur == pseudo:  
                item_pseudo = item
                if i == 1: animer_top3(tableau, item, "#FFD700", "#fff176")   # or qui clignote
                if i == 2: animer_top3(tableau, item, "#C0C0C0", "#eeeeee")  # argent qui clignote
                if i == 3: animer_top3(tableau, item, "#CD7F32", "#ffb07c")  # bronze qui clignote
        else:
            tableau.insert("", "end", values=(str(i), joueur, score))

    tableau.pack(padx=20, pady=20)
from playsound import playsound
import threading

def jouer_son_victoire():
    # Joue le son dans un thread séparé pour ne pas bloquer Tkinter
    threading.Thread(target=lambda: playsound("victory.mp3"), daemon=True).start()

def animer_top3(tableau, item, couleur1, couleur2, nb=6):
    """Fait clignoter une ligne du tableau TOP 3 + joue un son de victoire"""
    def toggle(n):
        if n > 0:
            actuel = tableau.item(item, "tags")
            if "blink1" in actuel:
                tableau.item(item, tags=("blink2",))
            else:
                tableau.item(item, tags=("blink1",))
            tableau.after(300, toggle, n-1)
    tableau.tag_configure("blink1", background=couleur1, foreground="black")
    tableau.tag_configure("blink2", background=couleur2, foreground="white")
    toggle(nb)
    jouer_son_victoire()
🌟✨ BRAVO [Pseudo] ! Tu es maintenant dans le TOP 3 !!! ✨🌟
if joueur == pseudo:  
    item_pseudo = item
    if i == 1: 
        animer_top3(tableau, item, "#FFD700", "#fff176")
        zone_chat.insert(tk.END, f"\n🌟✨ BRAVO {pseudo} ! Tu es PREMIER 🥇 !!! ✨🌟\n", "top3")
    if i == 2: 
        animer_top3(tableau, item, "#C0C0C0", "#eeeeee")
        zone_chat.insert(tk.END, f"\n🌟✨ BRAVO {pseudo} ! Tu es DEUXIÈME 🥈 !!! ✨🌟\n", "top3")
    if i == 3: 
        animer_top3(tableau, item, "#CD7F32", "#ffb07c")
        zone_chat.insert(tk.END, f"\n🌟✨ BRAVO {pseudo} ! Tu es TROISIÈME 🥉 !!! ✨🌟\n", "top3")

    zone_chat.see(tk.END)
zone_chat.tag_config("top3", foreground="#FFD700", font=("Comic Sans MS", 14, "bold"))
