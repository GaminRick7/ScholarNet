# ScholarNet: Research Paper Search and Visualization Platform

## Overview

ScholarNet is a sophisticated research paper search and visualization platform that combines advanced information retrieval techniques with interactive graph visualization. Built as part of CSC111 Winter 2025 Project 2, this application allows users to search through research papers using natural language queries and visualize the relationships between papers through an interactive citation network.

## Features

### 🔍 Advanced Search Capabilities
- **BM25 Information Retrieval**: Implements the BM25 ranking algorithm from scratch for highly relevant search results
- **Natural Language Processing**: Tokenizes and processes search queries with stop word removal
- **Citation-Aware Ranking**: Combines semantic similarity with citation count for optimal paper ranking
- **Search History**: Maintains a history of recent searches for quick access

### 📊 Interactive Visualization
- **Dynamic Graph Visualization**: Interactive D3.js-powered network visualization
- **Citation Network**: Shows papers as nodes and citations as edges
- **Node Weighting**: Node size reflects citation count and relevance
- **Interactive Features**: 
  - Drag and drop nodes
  - Hover to view paper titles
  - Click to fetch paper details

### 🎯 Advanced Filtering
- **Citation Count Filter**: Filter papers by minimum citation count (10+, 25+, 50+, 100+, 250+)
- **Author Filter**: Filter results by specific authors
- **Venue Filter**: Filter by publication venue
- **Real-time Filtering**: Apply multiple filters simultaneously

### 🔗 Paper Discovery
- **DOI Integration**: Automatic DOI lookup using Crossref API
- **Google Scholar Fallback**: Redirects to Google Scholar if DOI not found
- **Paper Details**: Access full paper information and links

## Architecture

### Core Components

#### 1. **Main Application (`main.py`)**
- Flask web server handling user interactions
- Route management for search, results, and DOI fetching
- Integration with search and visualization components

#### 2. **Search Engine (`search.py`)**
- **BM25 Class**: Implements the BM25 ranking algorithm
  - **Term Frequency (TF)**: `tf = (f × (k1 + 1)) / (f + k1 × (1 - b + b × (dl/avgdl)))`
  - **Inverse Document Frequency (IDF)**: `idf = log((N - df + 0.5) / (df + 0.5) + 1)`
  - **BM25 Score**: `score = Σ(idf × tf)` for each query term
  - **Configurable Parameters**: k1=1.25 (term frequency saturation), b=0.75 (length normalization)
- **Query Processing**: 
  - Tokenization with stop word removal (15 predefined stop words)
  - Case-insensitive text processing
  - Corpus preprocessing for efficient lookup
- **Ranking System**: 
  - **Weighted Scoring**: `final_score = 0.7 × normalized_bm25 + 0.3 × normalized_citations`
  - **Top-k Selection**: Returns top 200 papers by BM25, then top 75 by combined score
- **Graph Building**: 
  - **Level Assignment**: Direct matches (level 1), citations (level 2)
  - **Edge Creation**: Bidirectional citation relationships
  - **Subgraph Extraction**: O(k) complexity for k selected papers

#### 3. **Graph Management (`graph.py`)**
- **Graph Class**: 
  - **Adjacency List**: `dict[Any, _Vertex]` for O(1) vertex lookup
  - **Edge Storage**: `set[_Vertex]` for O(1) neighbor access
  - **Memory Efficient**: Only stores necessary vertex data
- **Vertex Class**: 
  - **Item Storage**: Paper object with metadata
  - **Neighbor Tracking**: Set-based adjacency for fast traversal
  - **Visibility Control**: Boolean flag for filtering
  - **Level Assignment**: Hierarchical graph structure (1=direct, 2=citation)
- **Paper Class**: 
  - **Immutable Data**: `@dataclass` with frozen=True
  - **Type Safety**: Strong typing for all fields
  - **Memory Layout**: Optimized for graph traversal
- **Data Loading**: 
  - **CSV Processing**: O(n) time complexity
  - **Edge Creation**: O(n²) worst case, O(n×avg_degree) average
  - **Error Handling**: Graceful handling of malformed data

#### 4. **Utilities (`utils.py`)**
- **Text Processing**: 
  - **Tokenization**: Regex-based word extraction with stop word filtering
  - **Stop Words**: 15 predefined academic stop words
  - **Normalization**: Case-insensitive processing
  - **Performance**: O(n) where n = text length
- **String Matching**: 
  - **Partial Matching**: Jaccard similarity with configurable threshold (default: 0.9)
  - **DOI Validation**: Cross-reference title matching for accuracy
  - **Fuzzy Matching**: Handles minor title variations
- **Weight Calculation**: 
  - **Citation Tiers**: 6-tier system (3, 5, 7, 10, 13, 16 weights)
  - **Non-linear Scaling**: Logarithmic-like progression for visual distinction
  - **Visual Optimization**: Node size proportional to citation impact

### Data Flow

1. **Data Loading**: Research papers loaded from CSV into graph structure
2. **Query Processing**: User query tokenized and processed
3. **BM25 Ranking**: Papers ranked by relevance to query
4. **Citation Integration**: Rankings combined with citation counts
5. **Graph Construction**: Relevant papers and their citations form network
6. **Visualization**: D3.js renders interactive network
7. **Filtering**: Real-time filtering based on user criteria

## Technology Stack

### Backend
- **Python 3.x**: Core application logic
- **Flask 3.1.0**: Web framework
- **Requests 2.32.3**: HTTP client for API calls

### Frontend
- **D3.js v7**: 
  - **Force Simulation**: Physics-based node positioning
  - **Drag Interactions**: Real-time node manipulation
  - **Event Handling**: Mouse events for user interactions
  - **SVG Rendering**: Scalable vector graphics for crisp display
- **Flowbite**: 
  - **Component Library**: Pre-built UI components
  - **Responsive Design**: Mobile-first approach
  - **Accessibility**: WCAG 2.1 compliant components
- **HTML5/CSS3**: 
  - **Semantic Markup**: Proper HTML structure
  - **CSS Grid/Flexbox**: Modern layout techniques
  - **Custom Styling**: Tailwind CSS integration

### APIs
- **Crossref API**: 
  - **REST Endpoint**: `https://api.crossref.org/works`
  - **Query Parameters**: Title and author-based search
  - **Response Format**: JSON with DOI and metadata
  - **Rate Limiting**: Respects API usage guidelines
  - **Error Handling**: Graceful fallback to Google Scholar
- **Google Scholar**: 
  - **Fallback Mechanism**: URL-encoded search queries
  - **Search Format**: `https://scholar.google.com/scholar?q={encoded_query}`
  - **Universal Access**: No API key required
  - **Broad Coverage**: Comprehensive academic search

### Testing & Quality
- **pytest**: Unit testing framework
- **python-ta**: Code quality and style checking
- **hypothesis**: Property-based testing

## Installation and Setup

### Prerequisites
- Python 3.8 or higher
- pip (Python package manager)

### Installation Steps

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd Project2-main-2
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Prepare data**
   - Ensure you have a `data/research-papers.csv` file with the following columns:
     - Paper ID
     - Authors (comma-separated)
     - Citation count
     - References (comma-separated)
     - Title
     - Venue
     - Additional metadata

4. **Run the application**
   ```bash
   python main.py
   ```

5. **Access the application**
   - Open your browser and navigate to `http://localhost:5000`

## Usage Guide

### Basic Search
1. Enter your search query in the search box
2. Click "Search" to find relevant papers
3. View the interactive graph visualization
4. Hover over nodes to see paper titles
5. Click on nodes to fetch paper details

### Advanced Filtering
1. Use the filter dropdowns to refine results:
   - **Citations**: Filter by minimum citation count
   - **Author**: Filter by specific authors
   - **Venue**: Filter by publication venue
2. Click "Apply Filter" to update the visualization

### Paper Discovery
1. Click on any paper node in the visualization
2. The system will attempt to find the paper's DOI
3. If found, you'll be redirected to the paper
4. If not found, you'll be redirected to Google Scholar

### Search History
- Recent searches are displayed in the search history panel
- Click on any previous search to quickly repeat it

## Data Format

The application expects a CSV file with the following structure:

```csv
paper_id,authors,citation_count,references,title,venue,additional_metadata,paper_id
```

Example:
```csv
1234,"['John Doe', 'Jane Smith']",25,"['5678', '9012']","Machine Learning Advances","ICML 2023","additional_data",1234
```

## Configuration

### BM25 Parameters (`search.py`)
```python
# Term frequency saturation parameter
k1: float = 1.25  # Range: 0.5-3.0, higher = more weight to term frequency

# Document length normalization parameter  
b: float = 0.75   # Range: 0.0-1.0, higher = more length normalization

# Scoring weights for final ranking
weight_sim: float = 0.7    # BM25 score weight
weight_cite: float = 0.3   # Citation count weight
```

### Visualization Settings (`templates/query.html`)
```javascript
// Force simulation parameters
const simulation = d3.forceSimulation(nodes)
    .force("link", d3.forceLink(links).id(d => d.id))
    .force("charge", d3.forceManyBody().strength(d => -d.weight * 23))
    .force("x", d3.forceX())
    .force("y", d3.forceY())

// Node sizing based on citation weight
.attr("r", d => d.weight * 3)

// Color scheme for different levels
const color = d3.scaleOrdinal(d3.schemeCategory10)
```

### Performance Tuning
- **Top-k Papers**: Adjust `get_top_n_paper_score()` limit (default: 200)
- **Final Results**: Modify `get_most_cited_score()` limit (default: 75)
- **Graph Depth**: Control citation network depth in `build_query_graph()`
- **Memory Limits**: Set maximum graph size for large datasets

## API Endpoints

### Core Routes
- `GET /`: 
  - **Purpose**: Home page with search interface
  - **Response**: HTML template with search form
  - **Performance**: Static content, fast loading

- `POST /search`: 
  - **Purpose**: Process search queries and redirect to results
  - **Parameters**: `query`, `citations_filter`, `author_filter`, `venue_filter`
  - **Response**: HTTP 302 redirect to `/results`
  - **Validation**: Required field validation

- `GET /results`: 
  - **Purpose**: Display search results with interactive visualization
  - **Parameters**: Query string parameters from search
  - **Response**: HTML with embedded JSON data for D3.js
  - **Data Format**: `nodesData` and `linksData` arrays

### Paper Discovery Routes
- `POST /fetch_doi`: 
  - **Purpose**: Fetch paper DOI and redirect to source
  - **Parameters**: `title`, `author`
  - **External Calls**: Crossref API → Google Scholar fallback
  - **Response**: HTTP 302 redirect to paper URL
  - **Error Handling**: 500 status for API failures

- `GET /loading`: 
  - **Purpose**: Loading page during paper fetching
  - **Parameters**: `title`, `author` for display
  - **Auto-submit**: JavaScript auto-submits DOI form
  - **UX**: Lottie animation for user feedback

## Contributing

### Code Style
- Follow PEP 8 guidelines
- Use type hints throughout
- Include docstrings for all functions
- Run `python-ta` for code quality checks

### Testing
- Write unit tests for new features
- Use `pytest` for test execution
- Maintain test coverage for core functionality

## Performance Considerations

### Computational Complexity
- **Graph Loading**: O(n²) for edge creation, O(n) for vertex creation
- **BM25 Search**: O(m × t) where m = papers, t = average tokens per paper
- **Graph Traversal**: O(k) for subgraph extraction where k = selected papers
- **Visualization Rendering**: O(n + e) where n = nodes, e = edges

### Memory Usage
- **Graph Storage**: O(n + e) space complexity
- **BM25 Model**: O(v × m) where v = vocabulary size, m = papers
- **Search Results**: O(k) for top-k papers
- **Visualization Data**: JSON serialization overhead

### Optimization Strategies
- **Lazy Loading**: Graph constructed on-demand
- **Caching**: BM25 scores cached for repeated queries
- **Pagination**: Limited results to prevent memory overflow
- **Compression**: Efficient data structures for large graphs

### Scalability Limits
- **Recommended Dataset Size**: < 10,000 papers for optimal performance
- **Memory Threshold**: ~500MB for typical academic datasets
- **Response Time**: < 2 seconds for standard queries
- **Concurrent Users**: Flask development server limits apply

## Future Enhancements

- **Advanced Analytics**: Citation impact metrics
- **Collaborative Filtering**: Paper recommendations
- **Export Features**: Graph export in various formats
- **Mobile Optimization**: Responsive design improvements
- **Real-time Updates**: Live data integration

## License

This project is part of CSC111 Winter 2025 coursework at the University of Toronto.

## Acknowledgments

- **D3.js**: For powerful data visualization capabilities
- **Flask**: For the web framework
- **Crossref**: For DOI lookup services
- **Flowbite**: For modern UI components

---

*ScholarNet: Connecting researchers through intelligent paper discovery and visualization.*

