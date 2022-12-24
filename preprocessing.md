## Text Preprocessing
The text preprocessing has to be carried-out in correct order. 

    1. Remove URL
    2. Remove HTML tag
    3. Expand contractions
    4. Remove punctutaions
    5. Spelling correction
    6. Grammar correction
    7. Remove Non-ASCII characters
## Packages Required
Install the following packages if not installed already:

    !pip install contractions
    !pip install language_tool_python
        
Import the following python packages for text preprocessing

    import re
    import nltk
    import spacy
    import string
    import contractions
    from textblob import TextBlob
    import language_tool_python
    
    PUNCT_TO_REMOVE = string.punctuation
    my_tool = language_tool_python.LanguageTool('en-US')
    
## Preprocess Function:
    def preprocess_txt(text):
      url_pattern = re.compile(r'https?://\S+|www\.\S+') 
      text =  url_pattern.sub(r'', text) #Remove URLs

      html_pattern = re.compile('<.*?>')
      text = html_pattern.sub(r'', text) #Remove HTML tags

      expanded_words = []
      for word in text.split():
        expanded_words.append(contractions.fix(word))
      text = ' '.join(expanded_words) #Expand contractions

      text = text.replace("-"," ")
      text = text.translate(str.maketrans('', '', PUNCT_TO_REMOVE)) #Remove punctuations
      pattern1= re.compile(r'[^0-9\w\s]') 
      text=pattern1.sub(r'', text)

      text = unicodedata.normalize('NFKD', text).encode('ascii', 'ignore').decode('utf-8', 'ignore')
      text = ''.join(i for i in text if ord(i)<128)
      my_matches = my_tool.check(text)  
      # defining some variables  
      myMistakes = []  
      myCorrections = []  
      startPositions = []  
      endPositions = []  

      # using the for-loop  
      for rules in my_matches:  
          if len(rules.replacements) > 0:  
              startPositions.append(rules.offset)  
              endPositions.append(rules.errorLength + rules.offset)  
              myMistakes.append(text[rules.offset : rules.errorLength + rules.offset])  
              myCorrections.append(rules.replacements[0])  

      # creating new object  
      my_NewText = list(text)   

      # rewriting the correct passage  
      for n in range(len(startPositions)):  
          for i in range(len(text)):  
              my_NewText[startPositions[n]] = myCorrections[n]  
              if (i > startPositions[n] and i < endPositions[n]):  
                  my_NewText[i] = ""  

      text = "".join(my_NewText)
      return text
