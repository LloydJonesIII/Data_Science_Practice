---
output:
  pdf_document: default
  html_document: default
---


# API Introduction Homework

## Student name:

### What is an API?
API (Application Programming Interface) is a set of rules that allows one software application to interact with another. A REST (Representational State Transfer) API is a type of web API that uses HTTP requests to get data such that the desired response can be obtained by engineering a web URL. In this assignment, you'll use GET requests to retrieve data from PubMed (in other words requests corresponding to a simple web link).

Often it is not much more than automating the process of clicking on a link in a browser and viewing the results!

## Part 1: PubMed API Data Retrieval (Detailed)

### Overview
In this part of the assignment, you will learn how to interact with the PubMed database programmatically using REST API calls to retrieve biomedical literature data. You'll then organize this data for subsequent analysis.

### What is PubMed?
PubMed is a free database maintained by the National Center for Biotechnology Information (NCBI) that contains more than 34 million citations and abstracts of biomedical literature. It's a primary resource for researchers in medicine, biology, and related fields.

### PubMed API Introduction
The NCBI provides access to PubMed through their "Entrez Programming Utilities" (E-utilities), which is a set of RESTful web services. We'll be using two main endpoints:
- **ESearch**: Used to search for articles and returns a list of article IDs (PMIDs)
- **EFetch**: Used to retrieve detailed information about specific articles using their PMIDs

### Understanding the Python Requests Package
Before diving into the PubMed API specifically, let's understand how to work with the requests package, which you'll use to make HTTP requests to the API.
Basic Example: Making a GET Request
Here's a simple example of how to make a GET request:

```
import requests

# Make a simple GET request to a URL
response = requests.get('https://api.example.com/data')

# Check if the request was successful
if response.status_code == 200:
    # Print the response content
    print(response.text)
    
    # If the response is JSON, you can parse it
    # data = response.json()
    # print(data)
else:
    print(f"Request failed with status code: {response.status_code}")
```

### (OPTIONAL) Register for an NCBI API key to increase your request limit:

One can increase the number of requests allowed by the NCBI by registering for an API key like so:
   - Visit https://www.ncbi.nlm.nih.gov/account/
   - Create an account and register for an API key
   - Store your API key securely in your application

### 1-A. Implementing the Search Function

Create a function to search for articles based on a query term. You'll need to construct a URL like this:

```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pubmed&term=machine+learning+in+healthcare+AND+(2020/01/01[Date+-+Publication]+:+2025/04/07[Date+-+Publication])&retmax=1000&retmode=json&sort=relevance
```

Example URL parameters explained:
- `db=pubmed`: Specifies the database to search
- `term=machine+learning+in+healthcare+AND+(2020/01/01[Date+-+Publication]+:+2025/04/07[Date+-+Publication])`: The search query with date filter
- `retmax=1000`: Maximum number of results to return
- `retmode=json`: Format of the response
- `sort=relevance`: Sorting method for results

Your function should:
1. Properly encode the search terms and date range
2. Handle HTTP request errors and implement retries
3. Parse the JSON response to extract the article IDs
4. Return a list of PMIDs (PubMed IDs)

Example response structure:
```json
{
  "esearchresult": {
    "count": "3592",
    "retmax": "10",
    "retstart": "0",
    "idlist": [
      "36773985",
      "37279863",
      "36986449",
      "36971333",
      "36902827",
      "36889641",
      "36746656",
      "36744646",
      "36732875",
      "36723480"
    ]
  }
}
```

### 1-B Saving Data in JSON format

Create a function that has a general python variable and a filename as input and saves the variable as a JSON file.
Use the json module and the `dump` method.
In the main part of your script this function should have as input the output from the previous execersise 1-A.


### 1-B Implementing the Data Retrieval Function (10 points)

Create a function to fetch detailed information for a single PubMed article ID. You'll need to construct a URL like this:

```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&id=36773985&retmode=xml&rettype=abstract
```
This is an example URL for retrieving the abstract information for a single article with PubMed ID 36773985.

Example URL parameters explained:
- `db=pubmed`: Specifies the database
- `id=36773985`: Comma-separated list of PMIDs
- `retmode=xml`: Format of the response (XML)
- `rettype=abstract`: Type of data to retrieve

Your function should:
1. Handle HTTP request errors and implement retries
2. Return the result as unparsed text string in XML format.

### 1-C. Implementing the XML Parser (15 points)

Create a function that has as input the text string obtained from the previous exercise and returns a dictionary with the following keys:

XML (eXtensible Markup Language) is a common format for API responses, especially in scientific and biomedical APIs like PubMed. Python's built-in xml.etree.ElementTree module provides a simple way to parse and navigate XML data.
Basic XML Structure
XML documents have a hierarchical tree structure with elements, attributes, and text content as shown here:

- PMID
- Article title
- Abstract (combine multiple sections if present)
- Publication date
- Journal name

Here is an example of the XML structure:

```xml
<PubmedArticleSet>
  <PubmedArticle>
    <MedlineCitation>
      <PMID>36773985</PMID>
      <Article>
        <ArticleTitle>Machine learning applications in healthcare: A comprehensive review</ArticleTitle>
        <Abstract>
          <AbstractText>This is the abstract text...</AbstractText>
          <AbstractText Label="METHODS">These are the methods...</AbstractText>
        </Abstract>
        <AuthorList>
          <Author>
            <LastName>Smith</LastName>
            <ForeName>John A</ForeName>
            <Affiliation>Department of Biomedical Informatics, University...</Affiliation>
          </Author>
          <!-- More authors -->
        </AuthorList>
        <Journal>
          <Title>Journal of Medical Informatics</Title>
          <!-- Journal info -->
        </Journal>
      </Article>
      <MeshHeadingList>
        <MeshHeading>
          <DescriptorName>Machine Learning</DescriptorName>
        </MeshHeading>
        <!-- More MeSH terms -->
      </MeshHeadingList>
    </MedlineCitation>
    <PubmedData>
      <PublicationStatus>ppublish</PublicationStatus>
      <ArticleIdList>
        <ArticleId IdType="pubmed">36773985</ArticleId>
        <ArticleId IdType="doi">10.1000/example-doi</ArticleId>
      </ArticleIdList>
    </PubmedData>
  </PubmedArticle>
  <!-- More articles -->
</PubmedArticleSet>
```

XML can be parsed using the ElementTree module in Python as in the example shown below.

Example data:
```
<root>
    <person id="1">
        <name>John Doe</name>
        <age>30</age>
        <skills>
            <skill>Python</skill>
            <skill>Data Science</skill>
        </skills>
    </person>
    <person id="2">
        <name>Jane Smith</name>
        <age>28</age>
        <skills>
            <skill>Machine Learning</skill>
            <skill>Statistics</skill>
        </skills>
    </person>
</root>
```

Example code:
```
import xml.etree.ElementTree as ET

# Parse the XML data from a string:
root = ET.fromstring(xml_data)

# Find all 'person' elements
persons = root.findall('person')
print(f"Found {len(persons)} person elements")

# Get the first person
first_person = root.find('person')
print(f"First person ID: {first_person.get('id')}")

# Find a specific element
name = first_person.find('name')
print(f"Name text: {name.text}")

# Find all skills of the first person
skills = first_person.findall('skills/skill')
print("Skills:")
for skill in skills:
    print(f"- {skill.text}")
```

Your parser should extract:
- PMID
- Article title
- Abstract (combine multiple sections if present)
- Publication date
- Journal name

Each article should be structured as a dictionary with these fields and the function 



### Useful API Documentation

To complete this assignment, you can consult:

1. PubMed E-utilities Documentation:
   https://www.ncbi.nlm.nih.gov/books/NBK25501/

2. Field descriptions for constructing complex queries:
   https://www.ncbi.nlm.nih.gov/books/NBK3827/#pubmedhelp.Search_Field_Descriptions

3. XML Response DTD (Document Type Definition):
   https://www.ncbi.nlm.nih.gov/books/NBK3828/#pubmedhelp.PubMed_XML_Element_Descriptions

### Tips for Success

1. Start by testing your API calls manually in a browser to understand the response format.
2. Implement and test each function separately before combining them.
3. Use proper error handling and debugging print statements.
4. Structure your code in a modular way for reusability.

### Submission

Submit the following:
1. Your Python script(s) with complete implementation
2. Sample output files (small subset, not all 1000 articles)
3. A brief report explaining your implementation and findings
