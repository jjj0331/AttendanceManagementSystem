# 1. 土台：Node.js環境
FROM node:20-alpine
WORKDIR /app
# 2. 設定ファイルをコピーしてライブラリをインストール
COPY package*.json ./
RUN npm install
# 3. ソースコードをコピー
COPY . .
# 4. 開発用サーバーを起動
CMD ["npm", "run", "dev"]