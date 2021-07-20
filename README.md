# tinyMl_classificationBlender
Classificação de velocidade e nível de líquido de um liquidificador usando dados de um acelerômetro, a plataforma Edge Impulse e a placa Arduino Nano 33 Ble sense.

# Data Collection
Iremos usar um liquidificador como “motor industrial” e nosso objetivo será determinar 4 combinações diferentes de nível de líquido e velocidade do liquidificador, além do estado “idle” e de anomalias com base nos dados do acelerômetro da nossa placa. O sistema foi montado de acordo com a foto abaixo:

![image](https://user-images.githubusercontent.com/74123993/126320944-fa86e316-1dfe-480a-a864-bb60b05ce7f2.png)

Algumas definições foram feitas:

   * nível “low”: nível de líquido no liquidificador baixo, o valor escolhido para representar esse nível foi de 250ml.
   
   ![image](https://user-images.githubusercontent.com/74123993/126321045-5c0a895b-8d36-439a-869a-fce6f3536348.png)
    
   * nível “full”: representa o nível do liquidificador cheio, sendo o valor numérico de 1 litro de líquido.

Com esses níveis definidos conectamos nossa placa à plataforma Edge Impulse e começamos o processo de aquisição dos dados para cada classe.

![image](https://user-images.githubusercontent.com/74123993/126321212-7731b8de-075b-4e75-a488-99c97a0793ae.png)

As classes escolhidas foram:
  * low_slow: nível baixo de líquido e velocidade 1(de 8) do liquidificador;
  * low_fast: nível baixo de líquido e velocidade 4(não foi utilizado o máximo para evitar vazamento quando estivermos usando o nível alto de líquido) do liquidificador;
  * high_slow: nível alto de líquido e velocidade 1 do liquidificador;
  * high_fast: nível alto de líquido e velocidade 4 do liquidificador.
  
![image](https://user-images.githubusercontent.com/74123993/126321387-c8c647fb-1213-4e17-9c94-b5f4c3204dec.png)

Foram coletados cerca de 6 minutos de dados do acelerômetro da nossa placa, e 5 classes.

![image](https://user-images.githubusercontent.com/74123993/126321771-fbfdc7b2-3844-4a13-bbba-6aac8c0143f1.png)

# Feature Extraction
Nosso “impulse” tem a seguinte forma:

![image](https://user-images.githubusercontent.com/74123993/126322077-560061ca-01a3-4611-98b4-eb08810bb288.png)

No explorador de características(features explorer) conseguimos observar que temos uma melhor “separação” das classes quando usamos os valores rms dos eixos x,y e z:

![image](https://user-images.githubusercontent.com/74123993/126322212-1e21824f-f626-42fe-902a-7ce9d724c2e0.png)

Para rede neural foram usadas 3 camadas com 20,20 e 10 perceptrons em cada camada, além de 50 epochs de treinamento:

![image](https://user-images.githubusercontent.com/74123993/126322334-8e535162-4786-4fc0-8e69-c6d0bdb55512.png)

Após o treinamento do nosso modelo, temos:

![image](https://user-images.githubusercontent.com/74123993/126322399-15fce171-abfa-4f5c-aee6-5f30e959d095.png)

Para a parte de detecção de anomalias iremos usar os parâmetros que melhor “separaram” nossas classes que foram determinados anteriormente:

![image](https://user-images.githubusercontent.com/74123993/126322526-3bff228c-bbb1-4a7b-9fd2-440fb038111e.png)

Obtemos os seguintes “clusters”:

![image](https://user-images.githubusercontent.com/74123993/126322583-3092ead3-373a-4f58-9d80-cc6ca7911218.png)

Quando testamos nosso modelo, obtemos:

![image](https://user-images.githubusercontent.com/74123993/126322698-bc10aaff-7144-4c91-837b-64fef97883d7.png)

Anteriormente foram usadas mais classes, mas o modelo não estava conseguindo distinguir bem as classes, então o número de classes foi reduzido de 8 para 4 classes.

# Deployment
Para o deploy da nossa placa foi feita uma alteração no código para que para cada classe tenhamos uma cor diferente no nosso led RGB:

![image](https://user-images.githubusercontent.com/74123993/126322788-d7ede9a1-5b62-4936-848e-c1804e63f177.png)

E caso tenhamos uma anomalia:

![image](https://user-images.githubusercontent.com/74123993/126322846-c8d5c41e-2866-4582-ad40-73d1cd40a941.png)

**obs: É importante lembrar que os leds são ligados com nível lógico baixo**

Portanto, teremos para as classes:
  * idle: Red+Green
  * full_fast: Red
  * full_slow: Red+Blue
  * low_slow: Blue
  * low_fast: Green
  * anomaly: Green+blue

Com isso foram feitos os testes com o modelo embarcado na placa:

  * low_slow:
  
  ![image](https://user-images.githubusercontent.com/74123993/126323319-acafea78-b783-454a-86ba-936c1a93bfe7.png)

  https://user-images.githubusercontent.com/74123993/126323884-09ac21c5-0d06-4f64-84a8-d52c4e8fe530.mp4

  * low_fast:
  
  ![image](https://user-images.githubusercontent.com/74123993/126324069-7b09c957-ce2f-4fbb-b4dd-6c9d2b3ebc96.png)
  
  Uploading low_fast.mp4…

  * full_slow:
  
  ![image](https://user-images.githubusercontent.com/74123993/126324250-42d231b5-1bd2-4b07-aac9-3ddcfd9bc794.png)
  
   https://user-images.githubusercontent.com/74123993/126324269-3a757d2d-a69a-4bc8-8077-4956eea57715.mp4

  * full_fast:
   
  ![image](https://user-images.githubusercontent.com/74123993/126324339-bc13dc79-b434-4218-afad-02cd50044033.png)

  Uploading full_fast.mp4…

  * idle e annomaly:
  
  Testado com o liquidificador parado:
  
  ![image](https://user-images.githubusercontent.com/74123993/126324574-5f8e2b2f-699e-4cdc-b560-41c2b48ac24f.png)

  Testado usando pedras de gelo junto com a água:

  ![image](https://user-images.githubusercontent.com/74123993/126324656-f029c85c-657b-4aa7-9b36-0e541aebad59.png)

  ![image](https://user-images.githubusercontent.com/74123993/126324724-35a0bfdb-47f7-475a-8385-6657da6fbced.png)

  O projeto pode ser encontrado no seguinte link:
  https://studio.edgeimpulse.com/public/40600/latest






