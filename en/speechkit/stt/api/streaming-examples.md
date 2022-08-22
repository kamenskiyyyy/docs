# Example uses for Streaming Recognition API v2

To try the examples in this section:

1. Clone the [{{ yandex-cloud }} API](https://github.com/yandex-cloud/cloudapi) repository:
   ```
   git clone https://github.com/yandex-cloud/cloudapi
   ```
1. [Find out the ID of the folder](../../../resource-manager/operations/folder/get-id.md) that your account has access to.
1. In the examples, an [IAM token](../../../iam/concepts/authorization/iam-token/) is used for authentication ([other authentication methods](../../concepts/auth.md)) Get an IAM token:
   * [Instructions](../../../iam/operations/iam-token/create.md) for a Yandex account.
   * [Instructions](../../../iam/operations/iam-token/create-for-sa.md) for a service account.
1. Download a [sample](https://storage.yandexcloud.net/speechkit/speech.pcm) audio file for recognition. The audio file is in [LPCM](https://en.wikipedia.org/wiki/Pulse-code_modulation) format with a sampling rate of 8000.

Then proceed to creating a client app.

{% list tabs %}

- Python 3

   1. Install the `grpcio-tools` package using the [pip](https://pip.pypa.io/en/stable/) package manager:

      ```bash
      pip install grpcio-tools
      ```

   1. Go to the directory hosting the [{{ yandex-cloud }} API](https://github.com/yandex-cloud/cloudapi/)repository, create an `output` directory, and generate the client interface code there:

      ```bash
      cd cloudapi
      mkdir output
      python -m grpc_tools.protoc -I . -I third_party/googleapis --python_out=output --grpc_python_out=output google/api/http.proto google/api/annotations.proto yandex/cloud/api/operation.proto google/rpc/status.proto yandex/cloud/operation/operation.proto yandex/cloud/ai/stt/v2/stt_service.proto
      ```

      As a result, the `stt_service_pb2.py` and `stt_service_pb2_grpc.py` client interface files as well as dependency files will be created in the `output` directory.

   1. Create a file (for example, `test.py`) in the root of the `output` directory and add the following code to it:

      ```python
      #coding=utf8
      import argparse

      import grpc

      import yandex.cloud.ai.stt.v2.stt_service_pb2 as stt_service_pb2
      import yandex.cloud.ai.stt.v2.stt_service_pb2_grpc as stt_service_pb2_grpc


      CHUNK_SIZE = 4000

      def gen(folder_id, audio_file_name):
          # Specify recognition settings.
          specification = stt_service_pb2.RecognitionSpec(
              language_code='ru-RU',
              profanity_filter=True,
              model='general',
              partial_results=True,
              audio_encoding='LINEAR16_PCM',
              sample_rate_hertz=8000
          )
          streaming_config = stt_service_pb2.RecognitionConfig(specification=specification, folder_id=folder_id)

          # Send the message with the recognition settings.
          yield stt_service_pb2.StreamingRecognitionRequest(config=streaming_config)

          # Read the audio file and send its contents in chunks.
          with open(audio_file_name, 'rb') as f:
              data = f.read(CHUNK_SIZE)
              while data != b'':
                  yield stt_service_pb2.StreamingRecognitionRequest(audio_content=data)
                  data = f.read(CHUNK_SIZE)

      def run(folder_id, iam_token, audio_file_name):
          # Establish a connection with the server.
          cred = grpc.ssl_channel_credentials()
          channel = grpc.secure_channel('stt.api.cloud.yandex.net:443', cred)
          stub = stt_service_pb2_grpc.SttServiceStub(channel)

          # Send data for recognition.
          it = stub.StreamingRecognize(gen(folder_id, audio_file_name), metadata=(('authorization', 'Bearer %s' % iam_token),))

          # Process server responses and output the result to the console.
          try:
              for r in it:
                  try:
                      print('Start chunk: ')
                      for alternative in r.chunks[0].alternatives:
                          print('alternative: ', alternative.text)
                      print('Is final: ', r.chunks[0].final)
                      print('')
                  except LookupError:
                      print('Not available chunks')
          except grpc._channel._Rendezvous as err:
              print('Error code %s, message: %s' % (err._state.code, err._state.details))


      if __name__ == '__main__':
          parser = argparse.ArgumentParser()
          parser.add_argument('--token', required=True, help='IAM token')
          parser.add_argument('--folder_id', required=True, help='folder ID')
          parser.add_argument('--path', required=True, help='audio file path')
          args = parser.parse_args()

          run(args.folder_id, args.token, args.path)
      ```
   1. Set the [folder ID](../../../resource-manager/operations/folder/get-id.md) and [IAM token](../../../iam/concepts/authorization/iam-token.md) and execute the created file. In the `path` argument, specify the path to the audio file to recognize:

      ```bash
      export FOLDER_ID=<folder ID>
      export IAM_TOKEN=<IAM token>
      python test.py --token ${IAM_TOKEN} --folder_id ${FOLDER_ID} --path speech.pcm

      Start chunk:
      alternative: Hello
      Is final: False

      Start chunk:
      alternative: Hello world
      Is final: True
      ```

- Node.js

   1. Go to the directory with the [{{ yandex-cloud }} API](https://github.com/yandex-cloud/cloudapi) repository, create a directory named `src`, and generate a dependency file named `package.json` in it:

      ```bash
      cd cloudapi
      mkdir src
      cd src
      npm init
      ```
   1. Install the necessary packages using npm:

      ```bash
      npm install grpc @grpc/proto-loader google-proto-files --save
      ```
   1. Download a gRPC [public key certificate](https://github.com/grpc/grpc/blob/master/etc/roots.pem) from the official repository and save it in the root of the `src` directory.
   1. Create a file, for example `index.js`, in the root of the `src` directory and add the following code to it:

      ```js
      const fs = require('fs');
      const grpc = require('grpc');
      const protoLoader = require('@grpc/proto-loader');
      const CHUNK_SIZE = 4000;

      // Get the folder ID and IAM token from the environment variables.
      const folderId = process.env.FOLDER_ID;
      const iamToken = process.env.IAM_TOKEN;

      //  Read the file specified in the arguments.
      const audio = fs.readFileSync(process.argv[2]);

      // Specifying the recognition settings.
      const request = {
          config: {
              specification: {
                  languageCode: 'ru-RU',
                  profanityFilter: true,
                  model: 'general',
                  partialResults: true,
                  audioEncoding: 'LINEAR16_PCM',
                  sampleRateHertz: '8000'
              },
              folderId: folderId
          }
      };

      // How often audio is sent in milliseconds.
      // or LPCM format, the frequency can be calculated using the formula: CHUNK_SIZE * 1000 / ( 2 * sampleRateHertz).
      const FREQUENCY = 250;

      const serviceMetadata = new grpc.Metadata();
      serviceMetadata.add('authorization', `Bearer ${iamToken}`);

      const packageDefinition = protoLoader.loadSync('../yandex/cloud/ai/stt/v2/stt_service.proto', {
          includeDirs: ['node_modules/google-proto-files', '..']
      });
      const packageObject = grpc.loadPackageDefinition(packageDefinition);

      // Establish a connection with the server.
      const serviceConstructor = packageObject.yandex.cloud.ai.stt.v2.SttService;
      const grpcCredentials = grpc.credentials.createSsl(fs.readFileSync('./roots.pem'));
      const service = new serviceConstructor('stt.api.cloud.yandex.net:443', grpcCredentials);
      const call = service['StreamingRecognize'](serviceMetadata);

      // Send a message with the recognition settings.
      call.write(request);

      // Read the audio file and send its contents in chunks.
      let i = 1;
      const interval = setInterval(() => {
          if (i * CHUNK_SIZE <= audio.length) {
              const chunk = new Uint16Array(audio.slice((i - 1) * CHUNK_SIZE, i * CHUNK_SIZE));
              const chunkBuffer = Buffer.from(chunk);
              call.write({audioContent: chunkBuffer});
              i++;
          } else {
              call.end();
              clearInterval(interval);
          }
      }, FREQUENCY);

      // Process server responses and output the result to the console.
      call.on('data', (response) => {
          console.log('Start chunk: ');
          response.chunks[0].alternatives.forEach((alternative) => {
              console.log('alternative: ', alternative.text)
          });
          console.log('Is final: ', Boolean(response.chunks[0].final));
          console.log('');
      });

      call.on('error', (response) => {
          // Handle errors
          console.log(response);
      });
      ```
   1. Set the [folder ID](../../../resource-manager/operations/folder/get-id.md) and [IAM token](../../../iam/concepts/authorization/iam-token.md) used in the script and run the created file. In the argument, specify the path to the audio file to recognize:

      ```bash
      export FOLDER_ID=<folder ID>
      export IAM_TOKEN=<IAM token>
      node index.js speech.pcm

      Start chunk:
      alternative:  Hello world
      Is final:  true
      ```

{% endlist %}