# Documentazione Tecnica: Configurazione Prompt PowerShell (Conda + Posh-Git)

Questa documentazione riassume la risoluzione del conflitto tra l'ambiente virtuale **Anaconda/Miniconda** e il modulo **posh-git** per PowerShell, garantendo la corretta coesistenza della visualizzazione dell'ambiente attivo e dello stato del repository Git nel prompt del terminale.

## 1. Descrizione del Problema
All'avvio di PowerShell, lo script di inizializzazione (`hook`) di Anaconda sovrascrive interamente la funzione nativa `prompt`. Se il modulo `posh-git` viene importato *dopo* o *all'interno* del ciclo di caricamento di Conda, quest'ultimo rileva una personalizzazione esistente e, per motivi di sicurezza, non applica le modifiche visive relative ai branch Git. Questo comportamento lascia visibile solo il prefisso dell'ambiente (es. `(base)`), nascondendo lo stato di Git.

## 2. Architettura della Soluzione
Per consentire a entrambi gli strumenti di coesistere, è necessario invertire rigorosamente l'ordine di caricamento all'interno del file di profilo di PowerShell (`$PROFILE`):
1. **Fase 1: Inizializzazione di posh-git:** Il modulo Git prepara la struttura del prompt.
2. **Fase 2: Inizializzazione di Conda:** Lo script di Conda si innesta sopra la struttura esistente, aggiungendo il modificatore dell'ambiente attivo (`(base)`) senza distruggere i metadati del repository.

---

## 3. Configurazione del Profilo (`$PROFILE`)

Il file di configurazione deve essere memorizzato nel percorso del profilo utente standard:
`C:\Users\Kael\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1`

(base) PS C:\Github\Aletheia-Agape> conda init powershell
no change     C:\Users\Profile\miniconda3\Scripts\conda.exe
no change     C:\Users\Profile\miniconda3\Scripts\conda-script.py
no change     C:\Users\Profile\miniconda3\condabin\conda.bat
no change     C:\Users\Profile\miniconda3\Library\bin\conda.bat
no change     C:\Users\Profile\miniconda3\condabin\_conda_activate.bat
no change     C:\Users\Profile\miniconda3\condabin\rename_tmp.bat
no change     C:\Users\Profile\miniconda3\condabin\conda_auto_activate.bat
no change     C:\Users\Profile\miniconda3\condabin\conda_hook.bat
no change     C:\Users\Profile\miniconda3\Scripts\activate.bat
no change     C:\Users\Profile\miniconda3\condabin\activate.bat
no change     C:\Users\Profile\miniconda3\condabin\deactivate.bat
no change     C:\Users\Profile\miniconda3\Scripts\activate
no change     C:\Users\Profile\miniconda3\Scripts\deactivate
no change     C:\Users\Profile\miniconda3\etc\profile.d\conda.sh
no change     C:\Users\Profile\miniconda3\etc\fish\conf.d\conda.fish
no change     C:\Users\Profile\miniconda3\shell\condabin\Conda.psm1
no change     C:\Users\Profile\miniconda3\shell\condabin\conda-hook.ps1
no change     C:\Users\Profile\miniconda3\Lib\site-packages\xontrib\conda.xsh
no change     C:\Users\Profile\miniconda3\etc\profile.d\conda.csh
- no change     C:\Users\Profile\Documents\WindowsPowerShell\profile.ps1 <= TO OPEN WITH NOTEPAD
No action taken.

### Codice Sorgente Definitivo

notepad C:\Users\Profile\Documents\WindowsPowerShell\profile.ps1

```powershell
# ==============================================================================
# CONFIGURAZIONE PROMPT: POSH-GIT & ANACONDA/MINICONDA
# ==============================================================================

# 1. Carica prima Posh-Git (prepara il terreno per il prompt di Git)
Import-Module posh-git

# 2. Subito dopo, lascia che Anaconda aggiunga il suo prefisso (base)
#region conda initialize
# !! Contents within this block are managed by 'conda init' !!
if (Test-Path "C:\Users\Kael\miniconda3\Scripts\conda.exe") {
    (& "C:\Users\Kael\miniconda3\Scripts\conda.exe" "shell.powershell" "hook") | Out-String | ?{$_} | Invoke-Expression
}
#endregion
```

---

## 4. Requisiti di Sicurezza e Criteri di Esecuzione
Se al riavvio del terminale PowerShell blocca il caricamento del profilo, è necessario aggiornare i criteri di esecuzione locali. Questo comando non ha alcun impatto sull'efficienza di calcolo hardware (GPU/CUDA) o sui framework ML:

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

---

## 5. Note per Ambienti di Sviluppo Avanzati (MLC-LLM & GPU)
* **Isolamento dell'Ambiente:** Questa configurazione agisce esclusivamente sullo strato di presentazione visiva (interfaccia CLI). 
* **Integrità CUDA/MLC:** Non vengono alterate le variabili d'ambiente di sistema relative a CUDA, cuDNN, o ai percorsi dei compilatori C++/Python necessari per i test sui modelli linguistici locali (LLM). L'allocazione della memoria VRAM e l'efficienza computazionale rimangono inalterate.
