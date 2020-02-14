---
title: "Huffman Coding"
date:  2020-02-12
layout: single
classes: wide
---

## Background and Motivation
Huffman Coding is a compression algorithm, 
which allocates the encoding length of any character based on the frequency of its appearance.  

### Naive Encoding
Any character in a text stored in memory as bits - sequences of 0s and 1s.
Let's say we don't know anything about the text, then we need a general way to encode each character which is unique and short as possible.
There are several common standards for encoding characters, let's take [ASCII][ascii_wiki] as an example.

ASCII attaches a running index number to each character. The character's encoding is the **binary representation of the index number**.
![an image alt text]({{site.baseurl}}/assets/huffman_coding/ascii_table_lge.png "ASCII table"){: .align-center}

- [Here][translation_table] is an interactive translation table
- [Here][ascii_disc] is a discussion about some pros and cons of the ASCII encoding

In terms of compression , this is not ideal: every character requires exactly 1 byte (=8 bits) of information. So, a character like "&", which is a very rare in a literary text, gets the same amount of bits as "e" which is extremely common.  
A text consisting of **N** characters will take up a space of **8\*N** bits in storage. 

### Simple example for frequency based encoding

Let's say we have the phrase "freeze geezer" and we want to encode it to bits.  
![an image alt text]({{site.baseurl}}/assets/huffman_coding/Frostillicus.gif "freeze geezer"){: .small .align-center}


Just for the sake of the example, let's make our own naive and frequency based encodings.  
Here is a naive 3 bit encoding:

| Character | Binary Encoding | 
|-------|--------|
| f | 000 | 
| r | 001 | 
| e | 010 | 
| z | 011 | 
|   | 100 |
| g | 101 | 
{: style="text-align: center;"} 

\*That empty cell is a space character (between the two words).
Note that in this example, we need at least 3 bits for each character to have a **unique** binary encoding.



"freeze geezer" becomes: 

`000 001 010 010 011 010 100 101 010 010 011 010 001`
{: style="text-align: center;"} 

This takes up (6+1+6)\*3 = 39 bits of information.  

Now, let's make a simple frequency based encoding:

| Character | Binary Encoding | 
|-------|--------|
| f | 1010 | 
| r | 110 | 
| e | 0 | 
| z | 111 |
|   | 1011 |
| g | 100 |  
{: style="text-align: center;"} 

This is still a unique encoding, don't worry about how exactly I got it just yet.  
Note that the length of bits allocated to any character is based on the frequency with which it appears in our phrase:  
- 'e' appears 6 times -> encoded to a short sequence - 0.
- 'f' appears only once- >  encoded to a long sequence - 1010 (longer than the naive encoding!).

"freeze geezer" becomes 

`1010 110 0 0 111 0 1011 100 0 0 111 0 110`
{: style="text-align: center;"} 

But this way only takes up only 29 bits of information! This is an imporvement of (1-29/35)\*100=17%.
This method will improve even further with longer phrases, and at worst will be as good as a naive encoding.

## Huffman Coding Algorithm Theory
To generalize the approach presented above, we need a systematic algorithm to derive our frequency based encoding. To make this as explicit as possible, I'm going to explain the algorithm steps with **text** compression in mind, this can be generalized to any file type. 
Also, to make this explanation a bit simpler, I'll refer to character frequency as the number of times a character appears in the text.
### Frequency count
We need to know the frequency of character in the text, the simplest way is to go through the text one character at a time and count the number of appearances
### Coding Tree
After we have the frequency dictionary, we construct a tree data structure, which organizes the characters by their frequency.  
Each node of the tree contains the **cumulative** frequency of all its children nodes.   
In addition, the bottom most node of any branch contains a character.

We organize the tree by pairing the nodes with the lowest frequency (n1, n2) and giving them a joint parent node (n3) which contains as a value the sum of frequencies (n3.frequency = n1.frequency + n2.frequency).
For the sake of consistency, we keep the node with the larger frequency as the right child (could also be the other way, the important thing is to keep a consistent rule).
Here is the Huffman tree for the "freeze geezer" example:

![an image alt text]({{site.baseurl}}/assets/huffman_coding/tree_example.jpg "Tree Example")

Note: 
1. Immediately  on the right of the root node is a bottom leaf with the 'e' char - which is the most frequent character.
2. The deepest nodes contain the characters 'f' and ' '(space) which are the least frequent
3. The root node contains the total cumulative frequency which is simply the amount of characters in the text - 13 in this case.


In the resulting structure, going from the root down to a bottom leaf, the **depth of each leaf is proportional to its frequency**.


### Encoding Dictionary
Travel the path from the root to the character, starting with an empty encoding for the charater
-  Going down the right branch we add a '0' to the encoding
-  Going down the left branch we add a '1' to the encoding

For example, to get to the 'f' character from the root we travel:  

`left(1) -> right(0) -> left(1) -> right(0)`  
{: style="text-align: center;"} 
Checking the encoding table in the example above , the encoding for 'f' is exactly '1010' !  
Note that because every character appears only once in the tree and there's a unique path to any character from the root, we get a unique frequency based encoding for each character! Perfect.

## Code Implementation
Here is the Python code implementing the steps described in the theory section
### Frequency Count
Input:  Data string (our text)
Output: Dictionary of the form **{character:frequency}**
{% highlight ruby %}
def GetFrequency(data):
    freqDict = {}
    for c in data:
        if c not in freqDict:
            freqDict[c] = 1
        else:
            freqDict[c]+=1
    freqDict = SortDict(freqDict)
    return freqDict
{% endhighlight %}

### Coding Tree
{% highlight ruby %}
def HuffmanCodingTree(freqDict):
    leafNodesList=[]
    internalNodesList=[]
    for k,v in freqDict.items():
        leafNodesList.append(Node(v,k))
    while leafNodesList or internalNodesList:
        if not leafNodesList and len(internalNodesList)==1:
            root = internalNodesList[0]
            return root
        candidates = []
        for i in internalNodesList[:2]:
            candidates.append([i,internalNodesList])
        for j in leafNodesList[:2]:
            candidates.append([j,leafNodesList])
        candidates.sort(key= lambda x: x[0].value)
        n1 = candidates[0][0]
        candidates[0][1].pop(0)
        n2 = candidates[1][0]
        candidates[1][1].pop(0)
        newNode = Node()
        newNode.Connect2(n1,n2)
        internalNodesList.append(newNode)
{% endhighlight %}
### Encoding Dictionary
{% highlight ruby %}
def MakeCodesDict(node,code,codesDict):
    if node.children == [None,None]:
        codesDict[node.char]=code
    else:
        MakeCodesDict(node.children[0],code+'0',codesDict)
        MakeCodesDict(node.children[1],code+'1',codesDict)
{% endhighlight %}


### Encoding and Decoding Text
Finally, we use the coding dictionary and tree to encode/decode any given text
{% highlight ruby %}
def EncodeText(text,codesDict):
    encoded = ''
    for c in text:
        encoded+=codesDict[c]
    return encoded

def DecodeText(text,hctRoot):
    decoded = ''
    currentNode = hctRoot
    for n in text:
        currentNode = currentNode.children[int(n)]
        if currentNode.children ==[None,None]:
            decoded+=currentNode.char
            currentNode = hctRoot
    return decoded
{% endhighlight %}


### Implementing the Code
Let's try some compression on a real text. You can download some free books from [Project Gutenberg][gutenberg].  
I picked "Heart of Darkness" by Joseph Conrad.
First of all, read the text and convert it to a binary (ascii) encoding
{% highlight ruby %}
f = open("heart_of_darkness.txt", "rb")
text = f.read()
f.close()
text = text.decode()
compList = []
textPartial = text[:15000]
binText = bin(reduce(lambda x, y: 256\*x+y, (ord(c) for c in textPartial), 0))
{% endhighlight %}

This is a pretty long text, to save some computing time I took only a part of it for comparison (first 15000 chars).  

Next, go through the steps described above to get the Huffman tree and encoded text
{% highlight ruby %}
freqDict = GetFrequency(textPartial)
hctRoot = HuffmanCodingTree(freqDict)
codesDict = {}
MakeCodesDict(hctRoot,'',codesDict)
huffmanText = EncodeText(textPartial,codesDict)
{% endhighlight %}

We can also check that the result is decodable back to the original text:
{% highlight ruby %}
decoded = DecodeText(huffmanText,hctRoot)
{% endhighlight %}
    
Last, check the compression percentage using the formula:
{% highlight ruby %}
compression = ( 1-(len(huffmanText)/len(binText)) )*100
{% endhighlight %}

We can run these steps for a different amount of characters, to see the effect of the length of the text on the compression rate.  
here is the resulting plot for char count from 1000 to 15000
![an image alt text]({{site.baseurl}}/assets/huffman_coding/comp_vs_chars.png "Compression vs Characters")

The compression somewhat improves with the length of the text, and seems to be asymptotic, reaching a limit value of about 43% at \~8000 chars. 




[ascii_wiki]: https://en.wikipedia.org/wiki/ASCII
[translation_table]: https://www.rapidtables.com/code/text/ascii-table.html
[ascii_disc]: https://bournetocode.com/projects/GCSE_Computing_Fundamentals/pages/3-3-5-ascii.html
[gutenberg][https://www.gutenberg.org/browse/scores/top]