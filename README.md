# Gestion-Utilisateur
# gestionnaire_utilisateurs.py
import json
import os

USERS_FILE = "users.json"

def reverse_hash(password: str) -> str:
    """Hachage simplifié : inverser la chaîne."""
    return password[::-1]

def load_users() -> list:
    if os.path.exists(USERS_FILE):
        try:
            with open(USERS_FILE, "r", encoding="utf-8") as f:
                return json.load(f)
        except Exception:
            print("Impossible de lire users.json — démarrage avec une liste vide.")
    return []

def save_users(users: list) -> None:
    try:
        with open(USERS_FILE, "w", encoding="utf-8") as f:
            json.dump(users, f, ensure_ascii=False, indent=2)
    except Exception as e:
        print("Erreur lors de la sauvegarde :", e)

def find_user_by_email(users: list, email: str):
    for u in users:
        if u.get("email", "").lower() == email.lower():
            return u
    return None

def add_user(users: list) -> None:
    nom = input("Nom : ").strip()
    email = input("Email : ").strip()
    if not nom or not email:
        print("Nom et email obligatoires.")
        return
    if find_user_by_email(users, email):
        print("Un utilisateur avec cet email existe déjà.")
        return
    mdp = input("Mot de passe : ")
    hashed = reverse_hash(mdp)
    users.append({"nom": nom, "email": email, "mdp": hashed})
    save_users(users)
    print("Utilisateur ajouté.")

def list_users(users: list) -> None:
    if not users:
        print("Aucun utilisateur.")
        return
    print("\nListe des utilisateurs :")
    for i, u in enumerate(users, start=1):
        print(f"{i}. Nom: {u.get('nom')} | Email: {u.get('email')}")
    print()

def modify_user(users: list) -> None:
    email = input("Email de l'utilisateur à modifier : ").strip()
    u = find_user_by_email(users, email)
    if not u:
        print("Utilisateur non trouvé.")
        return
    print("Laisser vide pour ne pas modifier.")
    new_nom = input(f"Nom ({u['nom']}) : ").strip()
    new_email = input(f"Email ({u['email']}) : ").strip()
    if new_email and new_email != u['email'] and find_user_by_email(users, new_email):
        print("Un utilisateur avec ce nouvel email existe déjà.")
        return
    new_mdp = input("Nouveau mot de passe (laisser vide = inchangé) : ")
    if new_nom:
        u['nom'] = new_nom
    if new_email:
        u['email'] = new_email
    if new_mdp:
        u['mdp'] = reverse_hash(new_mdp)
    save_users(users)
    print("Utilisateur modifié.")

def delete_user(users: list) -> None:
    email = input("Email de l'utilisateur à supprimer : ").strip()
    u = find_user_by_email(users, email)
    if not u:
        print("Utilisateur non trouvé.")
        return
    confirm = input(f"Confirmer suppression de {u['email']} ? (o/N) : ").strip().lower()
    if confirm == 'o':
        users.remove(u)
        save_users(users)
        print("Utilisateur supprimé.")
    else:
        print("Annulé.")

def login_user(users: list) -> None:
    email = input("Email : ").strip()
    mdp = input("Mot de passe : ")
    u = find_user_by_email(users, email)
    if not u:
        print("Identifiants incorrects.")
        return
    if u.get("mdp") == reverse_hash(mdp):
        print(f"Connexion réussie — bienvenue {u.get('nom')} !")
    else:
        print("Identifiants incorrects.")

def menu():
    users = load_users()
    while True:
        print("""
--- Gestionnaire d'utilisateurs ---
1 - Ajouter un utilisateur
2 - Liste des utilisateurs
3 - Modifier un utilisateur
4 - Supprimer un utilisateur
5 - Connexion
6 - Quitter
""")
        choix = input("Votre choix : ").strip()
        if choix == '1':
            add_user(users)
        elif choix == '2':
            list_users(users)
        elif choix == '3':
            modify_user(users)
        elif choix == '4':
            delete_user(users)
        elif choix == '5':
            login_user(users)
        elif choix == '6':
            print("Au revoir.")
            break
        else:
            print("Choix invalide.")

if __name__ == "__main__":
    menu()
