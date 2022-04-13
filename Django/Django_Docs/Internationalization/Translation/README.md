# Translation

## Overview

In order to make a Django project translatable, you have to add a minimal number of hooks to your Python code and templates. These hooks are called [translation strings](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Internationalization#translation-string). They tell Django: "This text should be translated into the end user's language, if a translation for this text is available in that language." It's your responsibility to mark translatable strings; the system can only translate strings it knows about.