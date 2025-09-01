# Progetto di Classificazione di Luoghi (Immagini e Video)

Questo progetto utilizza un modello di Deep Learning basato su `EfficientNetV2` per classificare immagini e video in base al luogo in cui sono stati ripresi. Il sistema è progettato per essere eseguito su Google Colab e integra un flusso di lavoro completo che va dall'addestramento del modello, all'inferenza su nuovi dati, fino alla valutazione finale delle performance.

## Caratteristiche Principali

-   **Transfer Learning**: Utilizza un modello `EfficientNetV2B0` pre-addestrato su ImageNet per un'elevata efficienza e accuratezza.
-   **Addestramento in Due Fasi**:
    1.  **Training della Testa**: Inizialmente viene addestrato solo il classificatore finale, mantenendo congelati i pesi della rete pre-addestrata.
    2.  **Fine-Tuning**: Successivamente, l'intero modello viene "scongelato" e addestrato con un learning rate molto basso per specializzarlo sul dataset specifico.
-   **Data Augmentation**: Applica trasformazioni casuali (rotazioni, zoom, flip) alle immagini di training per migliorare la robustezza del modello e prevenire l'overfitting.
-   **Processamento Video Avanzato**: Estrae frame a intervalli regolari dai video. Classifica ogni frame ed esegue un "voto di maggioranza" per determinare la classe dell'intero video. Seleziona e salva solo i frame migliori, filtrandoli in base a un **punteggio di nitidezza (sharpness)** per scartare le immagini mosse o sfocate.
-   **Logging e Tracciabilità**: Salva i log dettagliati dell'esecuzione in un file `run.log`. Tiene traccia dei file già processati in `processed.log` per evitare rielaborazioni inutili (anche se questa funzione è disabilitata di default nella fase di inferenza). Genera un file `results.csv` con i dettagli di ogni predizione.
-   **Metadati EXIF**: Aggiorna automaticamente i metadati EXIF delle immagini classificate, scrivendo la classe predetta nel campo "ImageDescription".


Per eseguire questo progetto, è necessario avere Python 3.9+ e le seguenti librerie. È consigliato utilizzare un ambiente virtuale (come `venv` o `conda`, oppure, finché si utilizza ancora Google Drive, va bene anche `Google Colab`) per gestire le dipendenze.

-   **Framework di Deep Learning principale:**
    -   `tensorflow~=2.15.0`

-   **Manipolazione di dati e array:**
    -   `pandas~=2.2.0`
    -   `numpy~=1.26.0`

-   **Elaborazione di immagini e video:**
    -   `opencv-python-headless~=4.9.0`
    -   `Pillow~=10.3.0`

-   **Utility:**
    -   `piexif~=1.1.3`
    -   `tqdm~=4.66.0`

## Struttura del Progetto

Per funzionare correttamente, il progetto richiede una specifica struttura di cartelle su Google Drive. Assicurati di crearla prima di eseguire il notebook.

-   `/content/drive/MyDrive/`
    -   `PROGETTO/`
        -   `dataset/`
            -   `train/`
                -   `classe_A/` (es. economia_interno)
                    -   `img1.jpg`
                    -   `...`
                -   `classe_B/` (es. stum_interno)
                    -   `img2.jpg`
                    -   `...`
            -   `val/`
                -   `classe_A/`
                -   `classe_B/`
            -   `test/`
                -   `classe_A/` **<-- ⚠️ ATTENZIONE: Le immagini devono essere qui**
                -   `classe_B/` **<-- e qui, non direttamente in /test**
        -   `ptp6.ipynb`


## Come Utilizzare il Notebook

Il notebook è diviso in sezioni logiche. Esegui le celle in ordine per completare l'intero ciclo di vita del modello.

1.  **Setup e Configurazione (Celle 1-5)**
    -   **Cella 1-4**: Installa le dipendenze (`piexif`), controlla la versione di Python, importa le librerie e monta Google Drive.
    -   **Cella 5**: Configura tutti i percorsi e gli iperparametri globali (dimensione delle immagini, batch size, epoche, ecc.). Verifica che il percorso `GDRIVE_PROJECT_PATH` corrisponda alla tua struttura su Drive. (Questa andrà modificata se si vorrà lavorare direttamente con il NAS)

2.  **Addestramento del Modello (Celle 6-9)**
    -   **Cella 6-8**: Definiscono tutte le funzioni necessarie per il training.
    -   **Cella 9**: Lancia il processo di addestramento.

3.  **Inferenza su Nuovi Dati (Celle 10-11)**
    -   **Cella 10**: Definisce la funzione `infer()`.
    -   **Cella 11**: Esegue l'inferenza sui file presenti nella cartella di test.

4.  **Valutazione del Modello (Celle 12-13)**
    -   **Cella 12**: Definisce la funzione `test()`.
    -   **Cella 13**: Esegue la valutazione finale del modello sul set di dati di test.


---

### Roadmap: Evoluzione del Progetto

Il progetto attuale è un prototipo funzionale. La visione a lungo termine è trasformarlo in un sistema di archiviazione fotografica intelligente e automatizzato, eseguito su un NAS, in grado di gestire una libreria con decine di categorie.

#### 1. Porting e Automazione (Esecuzione su NAS)

L'obiettivo è rendere il sistema autonomo, efficiente e indipendente da Google Drive/Colab.

-   **Refactoring da Notebook a Script (`.py`):**
    -   Suddividere il codice in file Python modulari (`train.py`, `infer.py`, `utils.py`).
    -   Utilizzare `argparse` per gestire i parametri da riga di comando.

-   **Configurazione Flessibile dei Percorsi:**
    -   Eliminare i percorsi fissi. I percorsi di input, del modello e la cartella "radice" della libreria fotografica devono essere passati come argomenti.

-   **Workflow "In-Place": Modifica Diretta dei Metadati (NON duplicare i file):**
    -   **Obiettivo chiave:** Rimuovere completamente la logica di copia dei file. Il sistema non dovrà creare una struttura di cartelle di "output" con le copie delle immagini.
    -   **Nuova Logica:** Lo script di inferenza dovrà:
        1.  Scandagliare la libreria fotografica.
        2.  Classificare ogni immagine.
        3.  **Scrivere la classe predetta direttamente nei metadati EXIF del file originale.** (il parametro soglia di sicurezza è già stato implementato, bisognerà aumentare la percentuale di sicurezza)
    - L'organizzazione delle foto potrà poi essere gestita da software di gestione fotografica (oppure dall'applicazione web creata appositamente) che possono leggere e filtrare in base ai tag EXIF.

-   **Automazione dei Processi (Scheduling):**
    -   Creare uno script "orchestratore" che analizza periodicamente la libreria, identifica i file non ancora taggati (magari controllando se il campo EXIF "ImageDescription" è vuoto) e li processa.
    -   Configurare un **cron job** sul NAS per eseguire lo script regolarmente (es. ogni notte).

#### 2. Scalabilità del Modello (Gestione di Molte più Classi)

Passare da 2 a decine di classi richiede un approccio più strutturato.

-   **Gestione di Dataset su Larga Scala:**
    -   Definire un processo rigoroso per la raccolta e l'etichettatura delle immagini per il training.

-   **Gestione dello Sbilanciamento delle Classi (Class Imbalance):**
    -   Utilizzare tecniche come **pesi di classe (`class_weight`)** o **campionamento** per gestire dataset dove alcune categorie hanno molte più immagini di altre.

-   **Architetture del Modello più Capienti:**
    -   Sperimentare con versioni più grandi di `EfficientNetV2` o architetture come **Vision Transformers (ViT)** per migliorare la capacità di distinguere tra un gran numero di classi.

-   **Classificazione Gerarchica (Approccio Avanzato):**
    -   Se le classi hanno una struttura logica (es. `Interni -> Casa -> Cucina`), addestrare un sistema di classificatori a cascata per migliorare l'accuratezza e la scalabilità.

#### 3. Usabilità e Interazione

Una volta che il sistema gira sul NAS, si può pensare a come interagire con esso.


-   **Interfaccia di Correzione e Feedback:**
    -   Sviluppare una semplice applicazione web ospitata sul NAS. L'interfaccia potrebbe permettere di:
        -   Visualizzare le foto e i tag predetti.
        -   **Correggere manualmente un tag errato.** Questa sarebbe una funzione interessante: le correzioni possono essere salvate per creare un dataset di "errori" da usare per ri-addestrare e migliorare il modello periodicamente (**Active Learning**).