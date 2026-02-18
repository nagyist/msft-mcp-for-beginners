# Selle näidise käivitamine

Soovitatav on paigaldada `uv`, kuid see ei ole kohustuslik, vaata [juhiseid](https://docs.astral.sh/uv/#highlights)

## -0- Loo virtuaalne keskkond

```bash
python -m venv venv
```

## -1- Aktiveeri virtuaalne keskkond

```bash
venv\Scripts\activate
```

## -2- Paigalda sõltuvused

```bash
pip install "mcp[cli]"
```

## -3- Käivita näidis


```bash
mcp run server.py
```

## -4- Testi näidist

Käivita server käsuga:

Lisa see *mcp.json* failile nii:

```json
```

Alusta serverit

Sisesta järgmine käsk:

```text
prompt
```

Peaksid nägema sarnast väljundit:

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vastutusest loobumine**:  
See dokument on tõlgitud AI tõlketeenuse [Co-op Translator](https://github.com/Azure/co-op-translator) abil. Kuigi püüame tagada täpsust, tuleb arvestada, et automaatsed tõlked võivad sisaldada vigu või ebatäpsusi. Originaaldokument oma emakeeles tuleks pidada autoriteetseks allikaks. Kriitilise teabe puhul soovitatakse kasutada professionaalset inimtõlget. Me ei vastuta selle tõlke kasutamisest tulenevate arusaamatuste või valesti mõistmiste eest.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->