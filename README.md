# Text-Document-Analyzer-Project
Written In C
this project parses a text document and utilizes a binary tree to generate
the following statistics about the file name entered by the user.
 1. average word length
 2. average sentence length
 3. frequency per 1000 words for each unique word provided in the document
 
 Note: all statistics are printed to a file

 Logic for analyzing a text document:
 1. A sentence in this program is defined by a question mark, period, and exclamation point.
 
 2. A word is defined as a sequence of characters that are alphabetic and a words end is defined by one of the characters in this string
 " \v\t()[],;:-\"@#$%^&*<>|~" and periods, question marks, and exclamation points. A letter is considered any alphabetic character
and excludes all others.

 Note: this program was originally used to formulate a guess as to weather James Madison, Thomas Jefferson or, John Jay
(writers of the federalist papers) wrote federalist 63. However, the logic that defined the statistics was somewhat different.
 Note: more comments explaining code and features coming soon!
 Note: concordance file is sent to the directory of the file that was analyzed
