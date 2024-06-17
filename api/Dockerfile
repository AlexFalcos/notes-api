# Creazione Applicazione che usa due Container
# Un Container per il DB con PostgreSQL (già esistente)
# Un Container che esegue l'appicazione con Express.js (creato da noi)

# Come eseguire il server del database PostgreSQL (immagine ufficale di Postgres)
# Per eseguire il Container con l'immagine di Postgres occorre
#   - una variavile d'ambiente per la password del db: POSTGRES_PASSWORD
#   - una variavile d'ambiente per il nome del db: POSTGRES_DB
# N.B.: Postgres di default è in ascolto nella porta 5432

# I db mantine i dati nella cartella interna ( es. "/var/lib/postgresql/data" per Postgre )
# Per non perdere il db con i dati, conviene montare il db su un Volume logico
# Creare un volume in Docker
# comando. docker volume create <nome volume>

# Step 1
# Creare il volume per conservare il db fuori dal container
# comando: docker volume create notes-db-data

# Eseguire il server del database
# comando: docker container run \
#              --detach \
#              --volume notes-db-data:/var/lib/postgresql/data \
#              --name=notes-db \
#              --env POSTGRES_DB=notesdb \
#              --env POSTGRES_PASSWORD=secret \
#              --network=notes-api-network \
#              postgres:12

# Analizzare il container
# comando: docker container inspect --format='{{range .Mounts}} {{ .Name }} {{end}}' notes-db
# Oppure
# comando: docker container inspect notes-db

# Accedere ai log dal container
# comando: docker container logs notes-db

# Creare una rete con Docker
# comando: docker network create notes-api-network

# Collegare un container alla rete creata
# comando: docker network connect notes-api-network notes-db

# Ispezionare un container
# comando: docker container inspect notes-db
# (Per capire ad esempio se è in esecuzione ed è connesso ad uno specifico network)

# Step 2
# Creazione di una immagine di Build Multi-Stadio
# Il primo stadio viene usato per il build e l'installazione delle dipendenze
# Il secondo stadio viene usato per eseguire l'applicazione

# Stage One
# Uso node (variante Alpine) e builder come nome dello stage
FROM node:lts-alpine as builder

# Installazione pacchetti per lo strumento "node-gyp" (python3 make g++)
RUN apk add --no-cache python3 make g++

# Imposta la cartella di lavoro predefinita con la nuova cartella /app
WORKDIR /app

# Copia il file package.json che contiene informazioni riguardo a tutte le dipendenze necessarie per l'applicazione
COPY ./package.json .

# Effettua la build dell'applicazione (--omit=dev per escludere dalla compilazione le parti di sviluppo)
RUN npm install --only=prod

# Stage Two
# Uso node (variante Alpine)
FROM node:lts-alpine

# Porta pubblicata
EXPOSE 3000

# Impostiamo la variabile di ambiente NODE_ENV su production
ENV NODE_ENV=production

# Utente di default (utente no-route chiamato node)
USER node

# Crea una cartella chiamata app all'interno della cartella home dell'utente node
RUN mkdir -p /home/node/app

# Imposta la cartella di lavoro predefinita con la nuova cartella /home/node/app
WORKDIR /home/node/app

# Copia il resto del contenuto della cartella corrente (.) del file system host nella cartella di lavoro (.) all'interno dell'immagine
COPY . .

# Copia la cartella node_modules dallo stadio builder (contiene le dipendeze installate per eseguire l'applicazione)
COPY --from=builder /app/node_modules  /home/node/app/node_modules

# Comando di default, avvia node
CMD [ "node", "bin/www" ]

# Build dell'immagine con tag
# comando: docker image build --tag notes-api .

# Esecuzione di un container con l'immagine eseguibile creata con tag (es. notes-api)
# comando: docker container run \
#               --detach \
#               --name notes-api \
#               --env DB_HOST=notes-db \
#               --env DB_DATABASE=notesdb \
#               --env DB_PASSWORD=secret \
#               --publish 3000:3000 \
#               --network=notes-api-network \
#               notes-api
# N.B.: La variabile d'ambiente DB_HOST è l'host del sever del db
#       Dato che sia il database del server che l'API sono collegati alla stessa rete bridge definita da un utente, 
#       il server del database può essere indicato usando il nome del suo container, che in questo caso è notes-db.
#       La variabile d'ambiente DB_DATABASE è il db che userà l'API 
#       Il nome è stato definito nell'esecuzione del container del server dalla variabile POSTGRES_DB, che in questo caso è notesdb.
#       La variabile d'ambiente DB_PASSWORD è la password per connettersi al db 
#       La password è stata definito sempre nell'esecuzione del container del server dalla variabile POSTGRES_PASSWORD, che in questo caso è secret.

# Eseguire un comando personalizzato all'interno di un container in esecuzione
# comando: docker container exec <identificatore container> <comando>
# Esempio per eseguire la migrazione del db, necessaria per configurare le tabelle del database
# comando: docker container exec notes-api npm run db:migrate

# Eseguire un comando interattivo all'interno di un container in esecuzione, occorre usarei l'opzione -it
# comando: docker container exec -it <identificatore container> <comando>
# Esempio di accesso alla shell, all'interno del container notes-api
# comando: docker container exec -it notes-api sh
