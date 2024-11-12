---
title: "Whisper"
layout: "default"
---

# Whisper `proto`

### TranscriptionRequest

Сообщение `TranscriptionRequest` используется для отправки аудио данных и настроек для транскрипции.

- **audio_data**: (тип: `bytes`) Аудио данные для транскрипции.
- **settings**: (тип: `Settings`) Кастомные настройки для процесса транскрипции.

```protobuf
message TranscriptionRequest {
    bytes audio_data = 1;
    Settings settings = 2;
}
```

### TranscriptionResponse

Сообщение `TranscriptionResponse` содержит результат транскрипции, включая транскрибированный текст и обнаруженные эмоции.

- **transcription**: (тип: `string`) Текст, полученный в результате транскрипции аудио данных.
- **emotion**: (тип: `string`) Обнаруженные эмоции в аудио данных, если применимо.

```protobuf
message TranscriptionResponse {
    string transcription = 1;
    string emotion = 2;
}
```

### Settings

Сообщение `Settings` позволяет настраивать процесс транскрипции с помощью различных параметров.

- **language**: (тип: `string`) Код языка для транскрипции (например, "en" для английского).
- **beam_size**: (тип: `google.protobuf.Int32Value`) Размер луча для поиска.
- **best_of**: (тип: `google.protobuf.Int32Value`) Количество лучших шагов, которые нужно рассмотреть.
- **patience**: (тип: `google.protobuf.FloatValue`) Параметр терпения для поиска с использованием луча, контролирующий завершение.
- **no_speech_threshold**: (тип: `google.protobuf.FloatValue`) Порог для обнаружения отсутствия речи в аудио.
- **temperature**: (тип: `google.protobuf.FloatValue`) Температура для семплирования при декодировании.
- **hallucination_silence_threshold**: (тип: `google.protobuf.FloatValue`) Порог для обнаружения воображаемой тишины в транскрипции.

```protobuf
message Settings {
    string language = 1;
    google.protobuf.Int32Value beam_size = 2;
    google.protobuf.Int32Value best_of = 3;
    google.protobuf.FloatValue patience = 4;
    google.protobuf.FloatValue no_speech_threshold = 5;
    google.protobuf.FloatValue temperature = 6;
    google.protobuf.FloatValue hallucination_silence_threshold = 7;
}
```

Эти настройки позволяют пользователям точно настраивать процесс транскрипции для удовлетворения специфических требований или оптимизации под различные условия записи.

# Whisper `proto_stt.py`

## TranscribeAudio

```python
def TranscribeAudio(self, request: stt_pb2.TranscriptionRequest, context):
        try:
            logger.info("Received audio file for transcription.")
            audio_data = np.frombuffer(request.audio_data, dtype=np.int16)
            audio_data = pcm2float(audio_data)
            logger.info(f"Original audio data length: {len(audio_data)}")
            audio_data = nr.reduce_noise(y=audio_data, sr=16000)

            settings = request.settings
            language = settings.language
            beam_size = settings.beam_size.value if settings.HasField('beam_size') else 5  # default value
            best_of = settings.best_of.value if settings.HasField('best_of') else 5
            patience = settings.patience.value if settings.HasField('patience') else 1.0
            no_speech_threshold = settings.no_speech_threshold.value if settings.HasField('no_speech_threshold') else 0.6
            temperature = settings.temperature.value if settings.HasField('temperature') else 0.2
         #   hallucination_silence_threshold = settings.hallucination_silence_threshold.value if settings.HasField('hallucination_silence_threshold') else 0.5

            logger.info(f"Settings - Language: {language}, Beam Size: {beam_size}, Best Of: {best_of}, Patience: {patience}, "
                        f"No Speech Threshold: {no_speech_threshold}, Temperature: {temperature}, "
                        f"Hallucination Silence Threshold:")

            # Transcribe audio with dynamic settings
            segments, _ = model.transcribe(
                audio_data,
                language=language,
                temperature=temperature,
                beam_size=beam_size,
                best_of=best_of,
                patience=patience,
                no_speech_threshold=no_speech_threshold,
              
            )
        
            transcription = " ".join([segment.text.strip() for segment in segments])
            logger.info(f"Transcription: {transcription}")

            # Predict emotion
            predicted_emotion = predict_emotion_from_numpy(audio_data, SAMPLE_RATE)
            logger.info(f"Predicted emotion: {predicted_emotion}")

            return stt_pb2.TranscriptionResponse(
                transcription=transcription,
                emotion=predicted_emotion
            )
        except Exception as e:
            logger.error(f"Error during transcription: {e}")
            context.set_details('An error occurred during transcription.')
            context.set_code(grpc.StatusCode.INTERNAL)
```

Как было указано выше функция TranscribeAudio создает транскрипцию на базе сырого pcm audio, далее идет конверсия, и так далее.

