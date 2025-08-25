import random
import time

# Génère un nombre aléatoire entre 1 et 100
nombre_secret = random.randint(1, 100)
tentatives = 0

# Blagues et phrases fun à utiliser
phrases_drôles = [
    "Est-ce que tu devines mieux que mon ex ? (Indice : pas dur 😅)",
    "Si tu trouves pas, t’es officiellement un NPC 💀",
    "Tu joues à quoi là ? À être mauvais ? 😂",
    "Devine encore... ou appelle ta mère pour t’aider 👀",
    "T’es plus paumé qu’un ado sans Wi-Fi.",
    "Tu veux un indice ? Bah non, c’est pas Noël 😌",
    "Essaie encore, t’es à ça d’être célèbre sur TikTok (ou pas).",
]

# Message d'accueil
print("Yo, bienvenue dans le jeu du siècle ! 🎉")
print("J'ai choisi un nombre entre 1 et 100. Si tu le trouves, t'es un giga boss.")
print("Mais si tu perds... t’auras juste perdu 3 minutes de ta vie. Let’s gooo ! 🚀")

while True:
    # Message marrant aléatoire
    print(random.choice(phrases_drôles))
    
    # L’utilisateur entre un nombre
    try:
        guess = int(input("Entre un nombre entre 1 et 100 : "))
    except ValueError:
        print("C’est pas un chiffre ça 🤡")
        continue

    tentatives += 1

    if guess < nombre_secret:
        print("Plus grand ! Comme ton ego après une victoire sur Fortnite.")
    elif guess > nombre_secret:
        print("Plus petit ! Comme ton skill à ce jeu 💀")
    else:
        print(f"🔥 GG ! T’as trouvé en {tentatives} tentative(s) !")
        print("Tu gagnes... absolument rien, mais t’as mon respect.")
        break

    time.sleep(1)  # Pause pour le style
