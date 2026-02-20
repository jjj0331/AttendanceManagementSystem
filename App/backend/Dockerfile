# ステージ1: ビルド用
FROM eclipse-temurin:17-jdk AS build
WORKDIR /app
# backend直下の全ファイルをコピー
COPY . .
# 実行権限を付けてビルドを実行
RUN chmod +x ./gradlew
RUN ./gradlew build -x test

# ステージ2: 実行用
FROM eclipse-temurin:17-jdk
WORKDIR /app
# ステージ1で出来上がったjarファイルをコピー（Gradleはbuild/libs/に出力されます）
COPY --from=build /app/build/libs/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]