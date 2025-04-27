# Install mkdocs using python

- Requires python & pip installed

## Windows & Vscode

### Create venv using code
ctrl + shift + p

"> Python: Create Virtual Environment"

Or directly in a terminal :
```
python -m venv venv
```

### Make sure ExecutionPolicy is set to Remote Signed
In at terminal, run this command :
```ps
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Activate venv
```ps
.\.venv\Scripts\activate
```

### Install mkdocs
```
pip install mkdocs-material
```

### Init mkdocs
```
mkdocs new .
```

### Preview config
Run a webserver on port 8000 with live reload
```
mkdocs serve
```

### Build the site
```
mkdocs build    
```

## Linux