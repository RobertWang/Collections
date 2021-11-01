> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [leetcode-cn.com](https://leetcode-cn.com/problems/implement-magic-dictionary/solution/cyu-yan-hashbiao-by-suckerfish-mini-9012/)

> 作者: suckerfish-mini 摘要: C 语言 hash 表来做

C 语言 hash 表来做

```
typedef struct {
    char key[101];
    UT_hash_handle hh;
} MagicDictionary;

MagicDictionary *g_user = NULL;

MagicDictionary* magicDictionaryCreate() {
    g_user = NULL;
    return NULL;
}

void magicDictionaryBuildDict(MagicDictionary* obj, char ** dictionary, int dictionarySize) {
    int len;
    MagicDictionary *temp = NULL;
    for (int i = 0; i < dictionarySize; i++) {
        temp = (MagicDictionary*)calloc(1, sizeof(MagicDictionary));
        strcpy(temp->key, dictionary[i]);
        HASH_ADD_STR(g_user, key, temp);
    }  
}

bool magicDictionarySearch(MagicDictionary* obj, char * searchWord) {
    int len = strlen(searchWord);
    for (int i = 0; i < len; i++) {
        char c = searchWord[i];
        for (int j = 0; j < 26; j++) {
            MagicDictionary *temp = NULL;
            if (('a' + j) != c) {
                searchWord[i] = 'a' + j;
            } else {
                continue;
            }
            HASH_FIND_STR(g_user, searchWord, temp);
            if (temp != NULL) {
                return true;
            }
        }
        searchWord[i] = c;
    }
    return false;
}

void magicDictionaryFree(MagicDictionary* obj) {
    MagicDictionary *cur = NULL;
    MagicDictionary *tar = NULL;
    HASH_ITER(hh, g_user, cur, tar) {
        if (cur) {
            HASH_DEL(g_user, cur);
            free(cur);
        }
    }
}
```