---
title: How to use pronunciation assessment in Speech Studio
titleSuffix: Azure Cognitive Services
description: The pronunciation assessment tool in Speech Studio gives you feedback on the accuracy and fluency of your speech, no coding required.
services: cognitive-services
author: sally-baolian
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: how-to
ms.date: 06/08/2022
ms.author: v-baolianzou
---

# Pronunciation assessment in Speech Studio

Pronunciation assessment provides subjective and objective feedback to language learners. Practicing pronunciation and getting timely feedback are essential for improving language skills. Assessments driven by experienced teachers can take a lot of time and effort, and makes a high-quality assessment expensive for learners. Pronunciation assessment can help make the language assessment more engaging and accessible to learners of all backgrounds. 

Pronunciation assessment provides various assessment results in different granularities, from individual phonemes to the entire text input. 
- At the full-text level, pronunciation assessment offers additional Fluency and Completeness scores: Fluency indicates how closely the speech matches a native speaker's use of silent breaks between words, and Completeness indicates how many words are pronounced in the speech to the reference text input. An overall score aggregated from Accuracy, Fluency and Completeness is then given to indicate the overall pronunciation quality of the given speech.  
- At the word-level, pronunciation assessment can automatically detect miscues and provide accuracy score simultaneously, which provides more detailed information on omission, repetition, insertions, and mispronunciation in the given speech.
- At the phoneme level, pronunciation assessment provides accuracy scores of each phoneme, helping learners to better understand the pronunciation details of their speech.

This article describes how to use the pronunciation assessment tool through the [Speech Studio](https://speech.microsoft.com). You can get immediate feedback on the accuracy and fluency of your speech without writing any code. For information about how to integrate pronunciation assessment in your speech applications, see [How to use pronunciation assessment](how-to-pronunciation-assessment.md).

## Try out pronunciation assessment

You can explore and try out pronunciation assessment even without signing in. 

> [!TIP]
> To assess more than 5 seconds of speech with your own script, sign in with an Azure account and use your Speech or Cognitive Services resource.

Follow these steps to assess your pronunciation of the reference text:

1. Go to **Pronunciation Assessment** in the [Speech Studio](https://aka.ms/speechstudio/pronunciationassessment).

1. Choose a supported [language](language-support.md#pronunciation-assessment) that you want to evaluate the pronunciation.

1. Choose from the provisioned text samples, or under the **Enter your own script** label, enter your own reference text.

   When reading the text, you should be close to microphone to make sure the recorded voice isn't too low. 

   :::image type="content" source="media/pronunciation-assessment/pa-record.png" alt-text="Screenshot of where to record audio with a microphone.":::

   Otherwise you can upload recorded audio for pronunciation assessment. Once successfully uploaded, the audio will be automatically evaluated by the system, as shown in the following screenshot.

   :::image type="content" source="media/pronunciation-assessment/pa-upload.png" alt-text="Screenshot of uploading recorded audio to be assessed.":::


## Pronunciation assessment results

Once you've recorded the reference text or uploaded the recorded audio, the **Assessment result** will be output. The result includes your spoken audio and the feedback on the accuracy and fluency of spoken audio, by comparing a machine generated transcript of the input audio with the reference text. You can listen to your spoken audio, and download it if necessary.

You can also check the pronunciation assessment result in JSON. The word-level, syllable-level, and phoneme-level accuracy scores are included in the JSON file. 

### Overall scores 

Pronunciation Assessment evaluates three aspects of pronunciation: accuracy, fluency, and completeness. At the bottom of **Assessment result**, you can see **Pronunciation score**, **Accuracy score**, **Fluency score**, and **Completeness score**. The **Pronunciation score** is overall score indicating the pronunciation quality of the given speech. This overall score is aggregated from **Accuracy score**, **Fluency score**, and **Completeness score** with weight.

:::image type="content" source="media/pronunciation-assessment/pa-display-score.png" alt-text="Screenshot of overall assessment scores." lightbox="media/pronunciation-assessment/pa-score.png":::

### Scores within words

### [Display](#tab/display)

The complete transcription is shown in the **Display** window. If a word is omitted, inserted, or mispronounced compared to the reference text, the word will be highlighted according to the error type. While hovering over each word, you can see accuracy scores for the whole word or specific phonemes. 

:::image type="content" source="media/pronunciation-assessment/pa-display-omission-zoom.png" alt-text="Screenshot of scores for a word and its phonemes." lightbox="media/pronunciation-assessment/pa-display-omission-full.png":::

### [JSON](#tab/json)

The complete transcription is shown in the `text` attribute. You can see accuracy scores for the whole word, syllables, and specific phonemes. You can get the same results using the Speech SDK. For information, see [How to use Pronunciation Assessment](how-to-pronunciation-assessment.md).

```json
{
    "text": "Today was a beautiful day. We had a great time taking a long long walk in the morning. The countryside was in full bloom, yet the air was crisp and cold towards end of the day clouds came in forecasting much needed rain.",
    "duration": 156100000,
    "offset": 800000,
    "json": {
        "Id": "f583d7588c89425d8fce76686c11ed12",
        "RecognitionStatus": 0,
        "Offset": 800000,
        "Duration": 156100000,
        "DisplayText": "Today was a beautiful day. We had a great time taking a long long walk in the morning. The countryside was in full bloom, yet the air was crisp and cold towards end of the day clouds came in forecasting much needed rain.",
        "SNR": 40.47014,
        "NBest": [
            {
                "Confidence": 0.97532314,
                "Lexical": "today was a beautiful day we had a great time taking a long long walk in the morning the countryside was in full bloom yet the air was crisp and cold towards end of the day clouds came in forecasting much needed rain",
                "ITN": "today was a beautiful day we had a great time taking a long long walk in the morning the countryside was in full bloom yet the air was crisp and cold towards end of the day clouds came in forecasting much needed rain",
                "MaskedITN": "today was a beautiful day we had a great time taking a long long walk in the morning the countryside was in full bloom yet the air was crisp and cold towards end of the day clouds came in forecasting much needed rain",
                "Display": "Today was a beautiful day. We had a great time taking a long long walk in the morning. The countryside was in full bloom, yet the air was crisp and cold towards end of the day clouds came in forecasting much needed rain.",
                "PronunciationAssessment": {
                    "AccuracyScore": 92,
                    "FluencyScore": 81,
                    "CompletenessScore": 93,
                    "PronScore": 85.6
                },
                "Words": [
                    // Words preceding "countryside" are omitted for brevity...
                    {
                        "Word": "countryside",
                        "Offset": 66200000,
                        "Duration": 7900000,
                        "PronunciationAssessment": {
                            "AccuracyScore": 30,
                            "ErrorType": "Mispronunciation"
                        },
                        "Syllables": [
                            {
                                "Syllable": "kahn",
                                "PronunciationAssessment": {
                                    "AccuracyScore": 3
                                },
                                "Offset": 66200000,
                                "Duration": 2700000
                            },
                            {
                                "Syllable": "triy",
                                "PronunciationAssessment": {
                                    "AccuracyScore": 19
                                },
                                "Offset": 69000000,
                                "Duration": 1100000
                            },
                            {
                                "Syllable": "sayd",
                                "PronunciationAssessment": {
                                    "AccuracyScore": 51
                                },
                                "Offset": 70200000,
                                "Duration": 3900000
                            }
                        ],
                        "Phonemes": [
                            {
                                "Phoneme": "k",
                                "PronunciationAssessment": {
                                    "AccuracyScore": 0
                                },
                                "Offset": 66200000,
                                "Duration": 900000
                            },
                            {
                                "Phoneme": "ah",
                                "PronunciationAssessment": {
                                    "AccuracyScore": 0
                                },
                                "Offset": 67200000,
                                "Duration": 1000000
                            },
                            {
                                "Phoneme": "n",
                                "PronunciationAssessment": {
                                    "AccuracyScore": 11
                                },
                                "Offset": 68300000,
                                "Duration": 600000
                            },
                            {
                                "Phoneme": "t",
                                "PronunciationAssessment": {
                                    "AccuracyScore": 16
                                },
                                "Offset": 69000000,
                                "Duration": 300000
                            },
                            {
                                "Phoneme": "r",
                                "PronunciationAssessment": {
                                    "AccuracyScore": 27
                                },
                                "Offset": 69400000,
                                "Duration": 300000
                            },
                            {
                                "Phoneme": "iy",
                                "PronunciationAssessment": {
                                    "AccuracyScore": 15
                                },
                                "Offset": 69800000,
                                "Duration": 300000
                            },
                            {
                                "Phoneme": "s",
                                "PronunciationAssessment": {
                                    "AccuracyScore": 26
                                },
                                "Offset": 70200000,
                                "Duration": 1700000
                            },
                            {
                                "Phoneme": "ay",
                                "PronunciationAssessment": {
                                    "AccuracyScore": 56
                                },
                                "Offset": 72000000,
                                "Duration": 1300000
                            },
                            {
                                "Phoneme": "d",
                                "PronunciationAssessment": {
                                    "AccuracyScore": 100
                                },
                                "Offset": 73400000,
                                "Duration": 700000
                            }
                        ]
                    },
                    // Words following "countryside" are omitted for brevity...
                ]
            }
        ]
    }
}
```

---



## Next steps

- Use [pronunciation assessment with the Speech SDK](how-to-pronunciation-assessment.md)
- Read the blog about [use cases](https://techcommunity.microsoft.com/t5/azure-ai-blog/speech-service-update-pronunciation-assessment-is-generally/ba-p/2505501)