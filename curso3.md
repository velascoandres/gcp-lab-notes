```shell script
echo "Creating App Engine app"
gcloud app create --region "us-central"

echo "Making bucket: gs://$DEVSHELL_PROJECT_ID-media"
gsutil mb gs://$DEVSHELL_PROJECT_ID-media

echo "Exporting GCLOUD_PROJECT and GCLOUD_BUCKET"
export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID
export GCLOUD_BUCKET=$DEVSHELL_PROJECT_ID-media

echo "Installing dependencies"
npm install -g npm@6.11.3
npm update

echo "Creating Datastore entities"
node setup/add_entities.js

echo "Project ID: $DEVSHELL_PROJECT_ID"
```

Agregar autentificacion

```html
<script src="https://www.gstatic.com/firebasejs/7.14.6/firebase-auth.js"></script>
```



Subscribirse a algun topic:


```shell script
gcloud pubsub subscriptions create cloud-shell-subscription --topic feedback
```



Publicar hacia un topic:

```shell script
gcloud pubsub topics publish feedback --message "Hello World"
```


Recuperar mensajes de un topic 

```shell script
gcloud pubsub subscriptions pull cloud-shell-subscription --auto-ack
```

Salida: 

```text
┌─────────────┬──────────────────┬────────────┬──────────────────┐
│     DATA    │    MESSAGE_ID    │ ATTRIBUTES │ DELIVERY_ATTEMPT │
├─────────────┼──────────────────┼────────────┼──────────────────┤
│ Hello World │ 1412102921014753 │            │                  │
└─────────────┴──────────────────┴────────────┴──────────────────┘
```


## Usar en `node.js`

```javascript
const {PubSub} = require('@google-cloud/pubsub');
```

Crear el cliente (Exportar como variable de entorno el ID del proyecto)

```shell script
export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID
```

```javascript
const pubsub = new PubSub({
  projectId: config.get('GCLOUD_PROJECT')
});
```

Obtener la referencia al topic:

```javascript
const feedbackTopic = pubsub.topic('feedback');
```

Publicar a un topic:

El topic debe ser convertido en buffer


```javascript
const dataBuffer=Buffer.from(JSON.stringify(feedback))
feedbackTopic.publish(dataBuffer);
```




### Languaje API


```javascript
const Language = require('@google-cloud/language');

const language = new Language.LanguageServiceClient({
    projectId: config.get('GCLOUD_PROJECT')
});


const document = {
        content: text,
        type: 'PLAIN_TEXT'
    };

language
    .analyzeSentiment({ document })
    .then(results => {
          const sentiment = results[0];
          // Get the sentiment score (-1 to +1)
          return sentiment.documentSentiment.score;
       });
```



```shell script
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
echo "Creating App Engine app"
gcloud app create --region "us-central"
echo "Making bucket: gs://$DEVSHELL_PROJECT_ID-media"
gsutil mb gs://$DEVSHELL_PROJECT_ID-media
echo "Exporting GCLOUD_PROJECT and GCLOUD_BUCKET"
export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID
export GCLOUD_BUCKET=$DEVSHELL_PROJECT_ID-media
echo "Installing dependencies"
npm install -g npm@6.11.3
npm update
echo "Installing Open API generator"
npm install -g api2swagger
echo "Creating Datastore entities"
node setup/add_entities.js
echo "Creating Cloud Pub/Sub topic"
gcloud pubsub topics create feedback
echo "Creating Cloud Spanner Instance, Database, and Table"
gcloud spanner instances create quiz-instance --config=regional-us-central1 --description="Quiz instance" --nodes=1
gcloud spanner databases create quiz-database --instance quiz-instance --ddl "CREATE TABLE Feedback ( feedbackId STRING(100) NOT NULL, email STRING(100), quiz STRING(20), feedback STRING(MAX), ratin
g INT64, score FLOAT64, timestamp INT64 ) PRIMARY KEY (feedbackId);"
echo "Enabling Cloud Functions API"
gcloud services enable cloudfunctions.googleapis.com
echo "Creating Cloud Function"
gcloud functions deploy process-feedback --runtime nodejs8 --trigger-topic feedback --source ./function --stage-bucket $GCLOUD_BUCKET --entry-point subscribe
echo "Project ID: $DEVSHELL_PROJECT_ID"
```




api2swagger -e https://8080-dot-13461995-dot-devshell.appspot.com/api/quizzes/places -o ./quiz-api.json