# Crypto Quest server

## Environment Setup

1. Install Rust from https://rustup.rs/
2. Install Solana CLI from https://docs.solana.com/cli/install-solana-cli-tools#use-solanas-install-tool
3. Install Spl-Token CLI from https://spl.solana.com/token
4. Install Anchor from https://project-serum.github.io/anchor/getting-started/installation.html
5. Install Postgres from https://www.postgresql.org/download/
6. Install Redis-server from https://redis.io/docs/getting-started/
7. Install Blender v3.1 from https://www.blender.org/download/releases/3-1/

## Backend server setup

### Prepare folders structure

```shell
$ mkdir CRYPTOQUESTNFT-SERVER
$ cd CRYPTOQUESTNFT-SERVER
$ mkdir metadata
$ mkdir blender_output
$ git clone https://github.com/anth226/cryptoquest-server.git server
```

### Update authtority keypair

Create 'keypair.json' file inside 'CRYPTOQUESTNFT-SERVER' folder. Paste inside 'keypair.json' secret key as Uint8Array format.

### Install dependencies for express server

```shell
$ cd server
$ npm install
```

### Update .env

1. Set LOCAL_ADDRESS for local ip address
2. Set UPDATE_AUTHORITY_DEVELOPMENT for PublicKey of update authtority for development collection
3. Set PINATA_API_KEY_DEVELOPMENT, PINATA_API_SECRET_KEY_DEVELOPMENT, PINATA_JWT_DEVELOPMENT, PINATA_GATEWAY_DEVELOPMENT for development Pinata account
4. Set BLENDER_OUTPUT_LOCAL_ADDRESS for for full path to 'blender_output' folder

## Database setup

1. Create database 'cryptoquest' with user 'postgres'

```shell
$ sudo -i -u postgres
$ psql
$ CREATE DATABASE cryptoquest;
$ \c cryptoquest
```

2. Change password for user postgres for new password from 'DB_PASSWORD' variable inside '/app/.env' file

```shell
$ ALTER USER postgres PASSWORD 'newpassword';
```

3. Execute all commands from './app/config/database.sql' file inside Postgres PSQL tool to create database and all tables

4. Add user for admin panel

```shell
$ INSERT INTO users (username, password) VALUES('admin', '$2a$08$HPXZtusLkfjdq6YbkyiDl.Kz9yi9izK.9FEXCzJ8.eXkn9UUIsjE2');
```

5. Fill out tables with tokens data

Woodland Respite Google Sheets - https://docs.google.com/spreadsheets/d/10-1C7PpMtKb8PZApgfGWHALCHjIPNPC6-Hrbo11zcUQ/edit?usp=sharing
Woodland Respite CSV - https://drive.google.com/file/d/1GB42BaKBtRb7B2_CR6UnmkmdjWVXBBB-/view?usp=sharing

Dawn of Man Google Sheets - https://docs.google.com/spreadsheets/d/1AN9TjHjJVLSnFaxjvZkPJC374k6HheXi4TlErD-_IsM/edit?usp=sharing
Dawn of Man CSV - https://drive.google.com/file/d/1gBrHVGl84Ig4MwrchyECO1QDoehg_Qs8/view?usp=sharing

Download csv files with data for tomes and copy them into '/tmp' folder on computer

Fill out tables with data from csv files

```shell
$ COPY woodland_respite(token_number, cosmetic_points, stat_points, ht_points_cp, ht_points_sp, ht_points_total, hero_tier, legacy) FROM '/tmp/Woodland_Respite_1250_Legacy.csv' DELIMITER ',' CSV HEADER;
$ COPY dawn_of_man(token_number, cosmetic_points, stat_points, ht_points_cp, ht_points_sp, ht_points_total, hero_tier, legacy) FROM '/tmp/Dawn_of_Man_1250_Legacy.csv' DELIMITER ',' CSV HEADER;
```

## Blender setup

1. Download Addon from https://drive.google.com/file/d/1BevZnrYjDIjIrwNyPAPixWLwWFvgYlfE/view?usp=sharing
   Reserved copy of Addon stored on VPS server (/home/zhopkins/Blender/Blender3/3.1/scripts/addons/CryptoQuest.zip)
2. Unzip Addon inside 'blender/3.1/scripts/addons/' with folder name 'CryptoQuest'

## Run backend server

```shell
// Run redis server inside separate terminal tab
$ redis-server
// Check is redis server working, should get ouput 'PONG'
$ redis-cli ping

$ npm run start:dev
```

## Bull dashboard

Bull dashboard available at http://localhost:5000/admin

## Create Candy Machine for development collection

### Setup solana wallet and config

```shell
$ cd ~/.config/solana/
$ solana-keygen new --outfile ~/.config/solana/devnet.json
$ solana config set --keypair ~/.config/solana/devnet.json
$ solana config set --url https://api.devnet.solana.com
$ solana airdrop 2 // Airdrop no more than 2 SOL per one transaction
$ solana balance // Verify balance after airdrop
$ solana config get // Verify keypair and solana cluster url
```

### Metaplex guide for creating CM2

Follow instructions on Metaplex website for creating Candy Machine v2 - https://docs.metaplex.com/candy-machine-v2/introduction

### Additional explanation for some points in Metaplex guide.

#### Following example creating 10 nfts.

Example of 'config.json' - https://drive.google.com/file/d/1OMez0ELcReEMedpN0k5m5q55i3SZkWsi/view?usp=sharing

Download script for generating metadata files - https://drive.google.com/file/d/1QqLLAVtAMw_zngg8qD9Ah3AgLmib7iSv/view?usp=sharing

Download Key gif for tokens - https://cryptoquest.mypinata.cloud/ipfs/QmNZV2SLmpdFC9VLTbJZbYEg3GdvJYjRUQ6Pp1yy38JajW

#### All following commands has to be executed inside 'metaplex/js/packages/cli/src' folder

```shell
$ mkdir upload
$ cp <path to generateJson.py> ./upload // Copy downloaded script into 'upload' folder
$ cd upload
$ python3 generateJson.py // Generate tokens metadata files
$ rm generateJson.py // Delete 'generateJson.py' file from upload folder after generating all metadata files
$ cp <path to Key gif image> ./ // Copy Key image inside 'upload' folder with name '0.gif'
$ for((i=0; i <=9; i++)); do cp 0.gif "${i}.gif"; done // Run loop to create images for quantity of tokens
```

Change sleep timeout for 5 seconds inside '/helpers/upload/pinata.ts' (from 'await sleep(500);' to 'await sleep(5000);'). Pinata is rate limited.

Upload collection to devnet

```shell
$ ts-node candy-machine-v2-cli.ts upload -e devnet -k ~/.config/solana/devnet.json -cp config.json -c example ./upload
```

Verify upload after upload will finish

```shell
$ ts-node candy-machine-v2-cli.ts verify_upload -e devnet -k ~/.config/solana/devnet.json -c example
```

After successfull upload new hidden folder '.cache' will be created.
Inside file 'devnet-example.json' will be candy machine id as "program.candyMachine".
Specify candy machine id inside .env file in 'cryptoquest-candy-machine-ui' repo.
