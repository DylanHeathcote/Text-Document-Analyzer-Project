/*this project written in C parses a text document and utilizes a binary tree to generate
 * the following statistics about the file name entered by the user.
 * 1. average word length
 * 2. average sentence length
 * 3. frequency per 1000 words for each unique word provided in the document
 *
 * Note: all statistics are printed to a file
 *
 * Logic for analyzing a text document:
 * 1.A sentence in this program is defined by a question mark, period, and exclamation point.
 *
 *2.A word is defined as a sequence of characters that are alphabetic and a words end is defined by one of the characters in this string
 * " \v\t()[],;:-\"@#$%^&*<>|~" and periods, question marks, and exclamation points. A letter is considered any alphabetic character
 * and excludes all others.
 *
 * Note:this program was originally used to formulate a guess as to weather James Madison, Thomas Jefferson or, John Jay
 * (writers of the federalist papers) wrote federalist 63. However, the logic that defined the statistics was somewhat different.
 *
 * Note: more comments explaining code and features coming soon!
 * Note: concordance file is sent to the directory of the file that was analyzed
 */


#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

typedef struct analysis{
    const char *Word;
    int wordCount;
    double wordfrequency;
    struct analysis *greaterThan;
    struct analysis *lessThan;
} Node;

//method definitions
void isFileOpenable(FILE *fPtr); //checks to see if the file selected is can be opened
void closeFilePointers(FILE *fPtr1, FILE *fPtr2);//closes file pointers
Node *Build_Tree(Node *node,char array[]); //tokenizes function, ads token to binary tree, prints all tokens to a file, and //helps with document statistics
Node *initialize_root(char * current_token); //initializes the root
Node *new_node(const char *Word);  //initializes a node to add to the tree
void add_word(Node *node, const char *current_token); //adds node or to the tree or increments the count
void print_statistics_to_file(Node* node,FILE *fPtr);  //prints binary tree to a file
void print_tree_to_file(Node* node,FILE *fPtr);//prints the tree to a concordance file
char* make_lowercase(char *str); // makes the string read by fscanf and makes it lowercase
void Gather_word_statistics(Node *node); //gathers both the number of words and the number of letters in the document
void Gather_remaining_statistics(Node *root);//function calls both Gather_frequency_statistics and Gather_Sentence_statistic
void Gather_frequency_statistics(Node *node);//calculates the frequency of each word in the document
void Gather_Sentence_statistic(char *current_token); //Calculates number of sentences in the document
int  Check_addable(char *current_token);//check to make sure the token can be added to the tree

//global variables that need to be accessed a lot
static int num_sentences=0;
static int num_letters_total=0;
static int num_words_total=0;

int main(){
    char line[81];
    Node *root=NULL;
    char filetoRead[80];


    printf("What file would you like to analyze:");
    scanf("%79s",filetoRead);
    FILE* fPtr=fopen(filetoRead, "r");

    isFileOpenable(fPtr);
    char* concordanceName=strcat(filetoRead,"_concordance.txt");
    FILE *fPtr3=fopen(concordanceName,"w+");


    while (fscanf(fPtr," %80[^\n]s",line)!=EOF){//O(n^2)
            root=Build_Tree(root, make_lowercase(line));
    }


    Gather_remaining_statistics(root);

    print_statistics_to_file(root,fPtr3);

    closeFilePointers(fPtr, fPtr3);

    return 0;
}


void isFileOpenable(FILE *fPtr){//O(1)
    if (fPtr==NULL) {
        printf("error cant open file");
        fclose(fPtr);
        exit(-1);
    }

}


void closeFilePointers(FILE *fPtr1, FILE *fPtr2){//O(1)
    fclose(fPtr1);
    fclose(fPtr2);
}


Node *Build_Tree(Node *node,char array[]){//O(n)

    char *current_token=strtok(array," \v\t()[],;:-\"@#$%^&*<>|~");

    int is_root=0;
    if (node == NULL) { //initialize the root
        node=initialize_root(current_token);
        is_root = 1;
    }

    while (current_token != NULL) {
        if (!is_root) {
            Gather_Sentence_statistic(current_token);
            if( Check_addable(current_token)) {
                add_word(node, current_token);
            }
        }

        current_token = strtok(NULL, " \t\v()[],;:-\"@#$%^&*,|~");
        is_root=0;
    }
    return node;
}


Node *initialize_root(char * current_token){//O(n)
    while(!Check_addable(current_token)){
        current_token=strtok(NULL," \v\t()[],;:-\"@#$%^&*<>|~");
    }

    Node *node = new_node(current_token);
    return node;
}


Node *new_node(const char *Word){//O(1)
    Node *node=malloc(sizeof(Node));
    node->Word=Word;
    node->wordCount=1;
    node->greaterThan=NULL;
    node->lessThan=NULL;
    node->wordfrequency=0.0;
    return node;
}


void add_word(Node *node, const char *current_token){//O(log(n))
    if(strcmp(current_token,node->Word)<0){
        if(node->lessThan!=NULL) {
            add_word(node->lessThan, current_token);
        }
        else {
            node->lessThan= new_node(current_token);
        }
    }
    else if(strcmp(current_token,node->Word)>0) {
        if(node->greaterThan!=NULL) {
            add_word(node->greaterThan, current_token);
        }
        else{
            node->greaterThan= new_node(current_token);
        }
    }
    else {
        ++node->wordCount;
    }
}


void print_statistics_to_file(Node *root,FILE *fPtr3){//O(n)
    fprintf(fPtr3,"num_words_total=%i\tnum_sentences=%i\t\tnum_letters_total=%i\n",num_words_total,num_sentences,num_letters_total);
    fprintf(fPtr3,"Average word length: %lf\naverage words per sentence: %lf\n", (double)num_letters_total/(double)num_words_total, (double)num_words_total/(double)num_sentences);
    print_tree_to_file(root,fPtr3);
}


void print_tree_to_file(Node* node,FILE *fPtr) {//O(n)
    if (node == NULL) {
        return;
    }
    print_tree_to_file(node->lessThan, fPtr);
    fprintf(fPtr, "Word: %-15s\t count: %-3i\tfrequency per thousand: %lf \n", node->Word, node->wordCount, node->wordfrequency);
    print_tree_to_file(node->greaterThan, fPtr);
}


char* make_lowercase(char *str){//O(n)
    char *lowerCase_str=strdup(str);
    for(int i=0;lowerCase_str[i]!='\0';i++){
        lowerCase_str[i]=tolower(lowerCase_str[i]);
    }
    return lowerCase_str;
}


void Gather_word_statistics(Node *node){//O(n^2)
    if(node==NULL) {
        return;
    }
    if(node->wordfrequency==0.0){
        num_words_total+=node->wordCount;
        int letterCount=0;
        for(int i=0;i< strlen(node->Word);i++){
            if(isalpha(node->Word[i])){
                letterCount++;
            }
        }
        num_letters_total+=node->wordCount*letterCount;

    }
    Gather_word_statistics(node->lessThan);
    Gather_word_statistics(node->greaterThan);
}


void Gather_remaining_statistics(Node *root) {//O(n)
    Gather_word_statistics(root);
    Gather_frequency_statistics(root);
}


void Gather_frequency_statistics(Node *node){//O(n)
    if(node==NULL) {
        return;
    }
    if(node->wordfrequency==0.0){
        node->wordfrequency=( (double) (node->wordCount) *1000)/(double)num_words_total;
    }
    Gather_frequency_statistics(node->lessThan);
    Gather_frequency_statistics(node->greaterThan);
}


void Gather_Sentence_statistic(char *current_token) {//O(n)

    for (int i = 0; i < strlen(current_token); i++) {
        if(isdigit(current_token[i])){ // if there is a list such as 1. 2. 3. it is not counted as a sentence
            current_token=NULL;
            return;
        }

        if (current_token[i] == '?' || current_token[i] == '!' || current_token[i] == '.') {
            current_token[i]='\0';
            num_sentences++;

        }
    }
}


int Check_addable(char *current_token){//O(n)
    for(int i=0;i<strlen(current_token);i++){   // if there is a list such as 1. 2. 3. it should not be counted and addeed as a word.
        if(isdigit(current_token[i])){
            return 0;
        }
    }
    if(strlen(current_token)==0 || *current_token=='.' || *current_token=='?' || *current_token=='!') {
        return 0;
    }
    return 1;
}
