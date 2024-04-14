# Address Classification Workflow

This document outlines the steps and methods employed in the address classification problem, including preprocessing, using BM25 for initial filtering, and post-processing to refine results.

## Preprocessing

The preprocessing steps include normalizing text data by removing unnecessary characters and stopwords, and standardizing abbreviations:

1. **Standardizing District or Town Names**: Convert abbreviations to their full forms.
   - Example: "TX Chu Se" → "Chu Se"

2. **Text Normalization**: Convert all text to lowercase and remove punctuation.
   - Example: “Trúc Bạch, Ba Đình, Hà Nội” → “truc bach ba đình ha noi”

3. **Stopword Removal**: Eliminate stopwords that do not add significant value to the address.
   - Example: “Quận 10, Thành phố Hồ Chí Minh” → “Quận 10 Hồ Chí Minh”

## Choosing Best Match Using BM25

We employ the BM25 algorithm to retrieve a preliminary list of candidate addresses that best match the query.

```python
def retrieve_candidates(query, k):
    scores = self.bm25.search(query)
    scores_index = np.argsort(scores)[::-1]
    top_results = np.array([self.addresses[i] for i in scores_index][:k])
    return top_results
```

## Refining Results with Edit Distance
After obtaining the initial candidates, we refine this list by comparing the edit distance between each candidate's address and the original query. The candidate with the smallest edit distance is chosen as the final match.

```python
    def refine_results(top_results, text):
        final_result = min(top_results, key=lambda result: editdistance.eval(result['full_address'].replace(",", ""), text))
        return final_result
```

## Post-Processing
In the final stage, we correct any remaining grammatical errors by comparing the structured elements of each address level in the result to the nearest addresses in our database. This ensures that the addresses are grammatically coherent and closely match the standard forms.

Edit Distance Utilization: The edit distance algorithm is used to find the closest match for each component of the address, correcting minor inaccuracies.
