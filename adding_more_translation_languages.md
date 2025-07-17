# Adding More Languages for Translation

## Overview

Your video transcription system uses AWS Translate for caption translation and supports transcription in multiple languages. This guide explains how to add more languages for both transcription and translation.

## Current System Architecture

### Translation Flow
1. **Transcription**: AWS Transcribe converts audio to text in the source language
2. **Translation**: AWS Translate converts captions from source language to target language
3. **Storage**: Translated captions are stored in S3 with language-specific naming

### Current Language Support

#### Transcription Languages (Source)
Currently supported in `source/web/templates/videos.hbs` (lines 55-69):
- Chinese (Simplified): `zh-CN`
- English variants: `en-AU`, `en-US`, `en-GB`, `en-IN`
- Spanish: `es-US`
- German: `de-DE`
- French: `fr-CA`, `fr-FR`
- Italian: `it-IT`
- Portuguese: `pt-BR`, `pt-PT`
- Indian languages: `ta-IN`, `te-IN`
- Asian languages: `ja-JP`, `ko-KR`
- Arabic: `ar-AE`, `ar-SA`

#### Translation Languages (Target)
Currently supported in `source/web/templates/video.hbs` (lines 109-119):
- Chinese: `zh` (Simplified), `zh-TW` (Traditional)
- English: `en`
- German: `de`
- French: `fr`
- Italian: `it`
- Japanese: `ja`
- Korean: `ko`
- Arabic: `ar`
- Portuguese: `pt`

## How to Add More Languages

### Step 1: Check AWS Service Support

Before adding languages, verify they are supported by AWS services:

#### AWS Transcribe Supported Languages
- Check the [AWS Transcribe documentation](https://docs.aws.amazon.com/transcribe/latest/dg/supported-languages.html)
- Languages must support real-time transcription if using streaming

#### AWS Translate Supported Languages
AWS Translate supports 75+ languages including:
- **European**: Albanian (sq), Bulgarian (bg), Croatian (hr), Czech (cs), Danish (da), Dutch (nl), Estonian (et), Finnish (fi), Greek (el), Hungarian (hu), Icelandic (is), Latvian (lv), Lithuanian (lt), Macedonian (mk), Maltese (mt), Norwegian (no), Polish (pl), Romanian (ro), Serbian (sr), Slovak (sk), Slovenian (sl), Swedish (sv), Ukrainian (uk)
- **Asian**: Bengali (bn), Gujarati (gu), Hindi (hi), Indonesian (id), Kannada (kn), Malayalam (ml), Marathi (mr), Punjabi (pa), Sinhala (si), Tamil (ta), Telugu (te), Thai (th), Urdu (ur), Vietnamese (vi)
- **African**: Afrikaans (af), Amharic (am), Hausa (ha), Somali (so), Swahili (sw)
- **Middle Eastern**: Armenian (hy), Azerbaijani (az), Georgian (ka), Hebrew (he), Persian (fa), Turkish (tr)

### Step 2: Add Transcription Languages

To add more source languages for transcription:

1. **Update the language dropdown** in `source/web/templates/videos.hbs`:
```html
<select id="languageSelection" class="form-select btn btn-success" aria-label="Default select example">
    <option selected value="{{defaultLanguage}}">{{defaultLanguage}}</option>
    <!-- Existing languages -->
    <option value="zh-CN">zh-CN</option>
    <option value="en-AU">en-AU</option>
    <!-- ... existing options ... -->
    
    <!-- Add new languages here -->
    <option value="sv-SE">sv-SE (Swedish)</option>
    <option value="nl-NL">nl-NL (Dutch)</option>
    <option value="pl-PL">pl-PL (Polish)</option>
    <option value="ru-RU">ru-RU (Russian)</option>
    <!-- Add more as needed -->
</select>
```

2. **Update language conversion logic** in `source/lambda/translatecaptions/translatecaptions.js`:
```javascript
function convertToTranslateLanguageCode(transcribeLanguageCode) {
  switch (transcribeLanguageCode) {
    case "zh-CN":
      return "zh";
    case "sv-SE":
      return "sv";
    case "nl-NL":
      return "nl";
    case "pl-PL":
      return "pl";
    case "ru-RU":
      return "ru";
    // Add more mappings as needed
    default:
      return transcribeLanguageCode.substring(0, 2);
  }
}
```

### Step 3: Add Translation Target Languages

To add more target languages for translation:

1. **Update the translation dropdown** in `source/web/templates/video.hbs`:
```html
<div class="dropdown-menu" aria-labelledby="translateDropdown">
    <!-- Existing languages -->
    <a class="dropdown-item" onclick="translateWithLanguageCode('zh')">Chinese (Simplified)</a>
    <a class="dropdown-item" onclick="translateWithLanguageCode('zh-TW')">Chinese (Traditional)</a>
    <!-- ... existing options ... -->
    
    <!-- Add new languages here -->
    <a class="dropdown-item" onclick="translateWithLanguageCode('sv')">Swedish</a>
    <a class="dropdown-item" onclick="translateWithLanguageCode('nl')">Dutch</a>
    <a class="dropdown-item" onclick="translateWithLanguageCode('pl')">Polish</a>
    <a class="dropdown-item" onclick="translateWithLanguageCode('ru')">Russian</a>
    <a class="dropdown-item" onclick="translateWithLanguageCode('hi')">Hindi</a>
    <a class="dropdown-item" onclick="translateWithLanguageCode('th')">Thai</a>
    <a class="dropdown-item" onclick="translateWithLanguageCode('vi')">Vietnamese</a>
    <!-- Add more as needed -->
</div>
```

### Step 4: Update Backend Configuration

No changes needed in the Lambda functions as they already support all AWS Translate languages. The `translatecaptions.js` function dynamically handles any language code passed to it.

### Step 5: Update Default Language Configuration

If you want to change the default language, update the backend configuration that returns `defaultLanguage` in the videos API response.

## Language Code Reference

### Common Language Codes for AWS Translate

| Language | AWS Translate Code | AWS Transcribe Code |
|----------|-------------------|---------------------|
| Swedish | `sv` | `sv-SE` |
| Dutch | `nl` | `nl-NL` |
| Polish | `pl` | `pl-PL` |
| Russian | `ru` | `ru-RU` |
| Hindi | `hi` | `hi-IN` |
| Thai | `th` | `th-TH` |
| Vietnamese | `vi` | `vi-VN` |
| Turkish | `tr` | `tr-TR` |
| Hebrew | `he` | `he-IL` |
| Greek | `el` | `el-GR` |
| Norwegian | `no` | `nb-NO` |
| Finnish | `fi` | `fi-FI` |
| Danish | `da` | `da-DK` |
| Czech | `cs` | `cs-CZ` |
| Hungarian | `hu` | `hu-HU` |
| Romanian | `ro` | `ro-RO` |
| Bulgarian | `bg` | `bg-BG` |
| Croatian | `hr` | `hr-HR` |
| Slovak | `sk` | `sk-SK` |
| Slovenian | `sl` | `sl-SI` |
| Estonian | `et` | `et-EE` |
| Latvian | `lv` | `lv-LV` |
| Lithuanian | `lt` | `lt-LT` |
| Ukrainian | `uk` | `uk-UA` |

## Testing New Languages

1. **Test transcription**: Upload a video in the new source language
2. **Test translation**: Verify translation works to/from the new language
3. **Check caption files**: Ensure translated captions are saved with correct naming (`videoId_languageCode.json`)
4. **Verify downloads**: Test downloading captions in different formats (VTT, SRT, TEXT)

## Deployment Considerations

### CloudFormation Template Updates
The deployment templates in `deployment/` may need updates if you're adding languages that require special configuration or regional considerations.

### Cost Implications
- AWS Transcribe: Charged per minute of audio
- AWS Translate: Charged per character translated
- More languages = potentially higher costs

### Performance Considerations
- Translation is done in batches with a pool limit of 10 concurrent requests
- Large volumes of text in certain languages may take longer to process

## Troubleshooting

### Common Issues

1. **Language not appearing in dropdown**: Check HTML template syntax
2. **Translation fails**: Verify language code is supported by AWS Translate
3. **Transcription fails**: Ensure language is supported by AWS Transcribe
4. **Wrong language detected**: AWS Transcribe auto-detection may need manual override

### Error Handling
The system includes error handling for:
- Unsupported language combinations
- Translation API failures
- Empty or invalid text content

## Best Practices

1. **Test thoroughly**: Always test new languages with real content
2. **Monitor costs**: Track usage for new languages
3. **User feedback**: Collect feedback on translation quality
4. **Regional considerations**: Some languages may work better in specific AWS regions
5. **Custom terminology**: Consider using AWS Translate Custom Terminology for domain-specific terms

## Example: Adding Swedish Support

Here's a complete example of adding Swedish:

1. **Add to transcription dropdown**:
```html
<option value="sv-SE">sv-SE (Swedish)</option>
```

2. **Add to translation dropdown**:
```html
<a class="dropdown-item" onclick="translateWithLanguageCode('sv')">Swedish</a>
```

3. **Update conversion function** (if needed):
```javascript
case "sv-SE":
  return "sv";
```

That's it! The system will automatically handle Swedish transcription and translation.