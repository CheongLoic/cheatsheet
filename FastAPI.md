# FastAPI Cheat Sheet

Apprenez rapidement à utiliser FastAPI avec ce guide pratique. 🚀

## **Installation**
Installez FastAPI et Uvicorn (un serveur ASGI rapide et léger) :
```bash
pip install fastapi uvicorn
```

---

## **Démarrage Rapide**
Créez un fichier `main.py` :
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

Lancez le serveur :
```bash
uvicorn main:app --reload
```
- `main`: nom du fichier.
- `app`: instance FastAPI.
- `--reload`: recharge automatique pendant le développement.

Swagger UI est accessible à : [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

---

## **Créer des Routes**
### Exemple de routes avec différentes méthodes HTTP :
```python
@app.post("/create")
def create_item(item: dict):
    return {"item": item}

@app.put("/update/{item_id}")
def update_item(item_id: int, item: dict):
    return {"item_id": item_id, "item": item}

@app.delete("/delete/{item_id}")
def delete_item(item_id: int):
    return {"item_id": item_id, "status": "deleted"}
```

---

## **Modèles avec Pydantic**
Utilisez Pydantic pour valider et typer les données :
```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float
    is_offer: bool = None

@app.post("/items/")
def create_item(item: Item):
    return {"item_name": item.name, "item_price": item.price}
```

---

## **Paramètres de Requête et de Chemin**
Combiner des paramètres de chemin et de requête :
```python
@app.get("/users/{user_id}")
def read_user(user_id: int, active: bool = True, limit: int = 10):
    return {"user_id": user_id, "active": active, "limit": limit}
```

---

## **Middleware**
Ajoutez des middlewares pour intercepter les requêtes/réponses :
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## **Gestion des Exceptions**
Gérer les erreurs personnalisées :
```python
from fastapi import HTTPException

@app.get("/items/{item_id}")
def read_item(item_id: int):
    if item_id < 1:
        raise HTTPException(status_code=400, detail="Item ID must be greater than 0")
    return {"item_id": item_id}
```

---

## **Dépendances**
Utilisez `Depends` pour réutiliser des logiques :
```python
from fastapi import Depends

def common_parameters(q: str = None, skip: int = 0, limit: int = 10):
    return {"q": q, "skip": skip, "limit": limit}

@app.get("/items/")
def read_items(commons: dict = Depends(common_parameters)):
    return commons
```

---

## **WebSocket**
Gérez les WebSockets avec FastAPI :
```python
from fastapi import FastAPI, WebSocket

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message reçu: {data}")
```

---

## **Tester avec TestClient**
Utilisez `TestClient` pour tester vos routes :
```python
from fastapi.testclient import TestClient

client = TestClient(app)

def test_read_root():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"Hello": "World"}
```

---

## **Authentification**
Exemple d'authentification OAuth2 :
```python
from fastapi.security import OAuth2PasswordBearer
from fastapi import Depends

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.get("/users/me")
def read_users_me(token: str = Depends(oauth2_scheme)):
    return {"token": token}
```

---

## **Structure Recommandée du Projet**
Organisez votre projet de manière claire :
```
my_project/
├── app/
│   ├── main.py
│   ├── models.py
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── users.py
│   │   ├── items.py
├── requirements.txt
```

Pour gérer les dépendances :
```bash
pip freeze > requirements.txt
pip install -r requirements.txt
```

---

Avec ce guide, vous êtes prêt à construire des API performantes avec FastAPI ! 🌟
