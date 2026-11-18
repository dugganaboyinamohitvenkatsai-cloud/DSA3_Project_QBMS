# Question Bank Management System (QBMS)
## Presentation & Comprehensive Viva Defense Guide
**Course:** Data Structures & Algorithms - 3 (25CS2103E) | **A.Y.:** 2026–2027  
**Team No:** 23 | **Section:** 8 | **Institution:** KLH Bachupally, Hyderabad  
**Guide:** Dr. V. Sireesha (Professor, Dept. of CSE)  
**Team Members:**  
- **D. Mohit Venkat Sai** (Roll No: `2520030106`)  
- **S. Prem Sai** (Roll No: `2520030561`)  

---

## 1. Presentation Speech (Word-for-Word Script)

### Step 1: Introduction (30 seconds)
> "Good morning respected evaluators and professors.  
> I am **D. Mohit Venkat Sai** (Roll No: `2520030106`), presenting alongside my teammate **S. Prem Sai** from **Team No. 23, Section 8**.  
> Under the guidance of **Dr. V. Sireesha**, we have developed our DSA-3 project: **The Question Bank Management System (QBMS)**."

### Step 2: Problem Statement & Motivation (45 seconds)
> "In academic institutions, educators manage large repositories of examination questions across multiple subjects, units, and Bloom's difficulty levels. Traditional systems suffer from three major bottlenecks:  
> 1. **Inefficient Retrieval:** Naive search scans every character sequentially ($O(N \times M)$), causing unacceptable delays in large question repositories.  
> 2. **Lack of Multi-Keyword Searching:** Educators cannot search for multiple syllabus concepts (e.g., 'Deadlock', 'Semaphore', and 'Paging') in a single pass.  
> 3. **Question Duplication & Typos:** Manual paper setting often leads to accidental duplicate or near-duplicate questions, reducing paper variety and quality.  
> Our project solves these problems by applying advanced string matching, tries, and dynamic programming."

### Step 3: Architecture & Algorithm Highlights (60 seconds)
> "We implemented our solution in two integrated layers: a **modular Java core backend** and a **feature-rich Web application with real-time algorithm visualizers**.  
> The core algorithmic intelligence is driven by four algorithms:  
> - **KMP (Knuth-Morris-Pratt):** Eliminates backtracking during pattern matching using a precomputed Longest Proper Prefix which is also Suffix (LPS) array, ensuring $O(N + M)$ linear-time search.  
> - **Rabin-Karp:** Leverages rolling polynomial hashing ($d=256$, prime $q=101$) to achieve average $O(N + M)$ pattern search with $O(1)$ sliding window transitions.  
> - **Aho-Corasick Multi-Pattern Automaton:** Combines a Trie prefix tree with BFS-constructed failure and dictionary links to search multiple syllabus keywords simultaneously in a single pass ($O(N + M_{total})$).  
> - **Levenshtein Distance (Dynamic Programming):** Calculates minimum edit operations (insertions, deletions, substitutions) to provide typo-tolerant fuzzy search and automated duplicate question detection with similarity percentage scoring.  
> Furthermore, we have an **Automated Question Paper Generator** that constructs balanced examination papers matching unit-wise weightage and difficulty constraints."

### Step 4: Conclusion & Transition to Live Demo (15 seconds)
> "By transforming examination management with formal algorithmic foundations, QBMS improves retrieval efficiency and ensures academic integrity.  
> We would now like to demonstrate our system and the algorithm visualizers."

---

## 2. In-Depth Algorithmic Breakdown

### 1. Knuth-Morris-Pratt (KMP) Algorithm
* **File Reference:** `KMP.java` & `algorithms.js`
* **Purpose:** Exact single-pattern substring search inside question text, subject, and topic names.
* **Why KMP over Naive Search?**
  * The naive algorithm compares pattern characters one by one. Upon mismatch, it resets the pattern pointer to `0` and increments the text pointer by only `1`. This causes worst-case time complexity of $O(N \times M)$ on repetitive texts (e.g., matching `"AAAAAB"` in `"AAAAAAAAAAAA"`).
  * KMP **never backtracks the text index $i$**. It moves strictly forward from $0$ to $N-1$. When a mismatch occurs at pattern index $j$, KMP uses the precomputed `LPS` array to skip redundant comparisons.
* **The LPS Array:**
  * `LPS[i]` stores the length of the longest proper prefix of `pattern[0...i]` that is also a suffix of `pattern[0...i]`.
  * Proper prefix means a prefix that is not the entire string itself.
  * Example: Pattern `"ABABC"`
    * `LPS[0] = 0` (`"A"`)
    * `LPS[1] = 0` (`"AB"`)
    * `LPS[2] = 1` (`"ABA"` prefix `"A"` == suffix `"A"`)
    * `LPS[3] = 2` (`"ABAB"` prefix `"AB"` == suffix `"AB"`)
    * `LPS[4] = 0` (`"ABABC"`)
* **Complexity:**
  * **Preprocessing Time:** $O(M)$ (building LPS array).
  * **Search Time:** $O(N)$ (scanning text).
  * **Overall Time Complexity:** $\mathbf{O(N + M)}$.
  * **Auxiliary Space:** $\mathbf{O(M)}$ for the LPS array.

---

### 2. Rabin-Karp Algorithm
* **File Reference:** `RabinKarp.java` & `algorithms.js`
* **Purpose:** Fast pattern matching using hash values.
* **How It Works:**
  1. Computes the hash value of the pattern of length $M$.
  2. Computes the hash value of the first substring window of length $M$ in the text.
  3. If hash values match, it performs a character-by-character check to guard against **spurious hits** (hash collisions).
  4. Slides the window by 1 position using a **Rolling Hash**:
     $$\text{hash}_{\text{next}} = \left( d \cdot (\text{hash}_{\text{prev}} - \text{text}[i] \cdot h) + \text{text}[i+M] \right) \pmod q$$
     - Base radix $d = 256$ (ASCII alphabet size)
     - Modulus $q = 101$ (a prime number to minimize collisions)
     - $h = d^{M-1} \pmod q$
* **Complexity:**
  * **Average / Best Time Complexity:** $\mathbf{O(N + M)}$.
  * **Worst-Case Time Complexity:** $O(N \times M)$ (if bad hash collisions occur for every window).
  * **Auxiliary Space:** $\mathbf{O(1)}$ (constant extra memory).

---

### 3. Aho-Corasick Algorithm
* **File Reference:** `AhoCorasick.java` & `algorithms.js`
* **Purpose:** Multi-pattern dictionary matching. Searching for multiple keywords (e.g., `["CPU", "Scheduling", "Deadlock", "Semaphore"]`) across questions in a single operation.
* **Why Not Run KMP for Each Pattern?**
  * If there are $K$ keywords, running KMP $K$ times takes $O(K \cdot (N + M))$ time.
  * Aho-Corasick processes **all $K$ patterns simultaneously in a single pass of the text**!
* **Three Structural Components:**
  1. **Trie (Go-to function):** Builds a prefix tree from all patterns.
  2. **Failure Links (Fail function):** Computed using Breadth-First Search (BFS). If a character does not match the current Trie branch, the failure link redirects to the state representing the longest proper suffix of the current prefix.
  3. **Output Links (Dict function):** Collects all matching pattern IDs associated with the current state and its suffix states.
* **Complexity:**
  * **Construction Time:** $O(M_{\text{total}} \times \Sigma)$ where $M_{\text{total}}$ is total length of all patterns.
  * **Matching Time:** $\mathbf{O(N + Z)}$ where $Z$ is the count of matches found.
  * **Total Time Complexity:** $\mathbf{O(N + M_{\text{total}} + Z)}$.
  * **Auxiliary Space:** $O(M_{\text{total}} \times \Sigma)$.

---

### 4. Levenshtein Distance (Dynamic Programming)
* **File Reference:** `Levenshtein.java` & `algorithms.js`
* **Purpose:** Fuzzy search (handling user typos) and duplicate question detection.
* **Definition:** The minimum number of single-character edits required to transform string $A$ into string $B$:
  1. **Insertion** of a character (cost = 1)
  2. **Deletion** of a character (cost = 1)
  3. **Substitution** of a character (cost = 1 if different, cost = 0 if identical)
* **DP Recurrence Relation:**
  For a 2D matrix $dp[i][j]$ where $i$ is prefix length of $A$ and $j$ is prefix length of $B$:
  $$dp[i][j] = \begin{cases} 
  i & \text{if } j = 0 \\
  j & \text{if } i = 0 \\
  dp[i-1][j-1] & \text{if } A[i-1] == B[j-1] \\
  1 + \min\begin{cases} dp[i-1][j] & \text{(Deletion)} \\ dp[i][j-1] & \text{(Insertion)} \\ dp[i-1][j-1] & \text{(Substitution)} \end{cases} & \text{otherwise}
  \end{cases}$$
* **Similarity Metric:**
  $$\text{Similarity \%} = \left( 1 - \frac{\text{Levenshtein Distance}(A, B)}{\max(\text{length}(A), \text{length}(B))} \right) \times 100$$
  - If Similarity $\ge 75\%$, the system flags the question as a **Potential Duplicate**.
* **Complexity:**
  * **Time Complexity:** $\mathbf{O(N \times M)}$ where $N = |A|$ and $M = |B|$.
  * **Space Complexity:** $\mathbf{O(N \times M)}$ (tabulation matrix).

---

## 3. Data Structures Inventory

| Data Structure | Implementation in Project | File |
| :--- | :--- | :--- |
| **Dynamic Array** | `ArrayList<Question>` to store the in-memory repository | `Main.java` |
| **Integer Array** | `int[] lps` table storing prefix-suffix lengths | `KMP.java` |
| **Trie / Tree** | Node objects with character transitions and suffix links | `AhoCorasick.java` |
| **Queue** | `Queue<Node>` / `LinkedList` for BFS failure link construction | `AhoCorasick.java` |
| **2D DP Grid** | `int[][] dp` matrix for edit distance tabulation | `Levenshtein.java` |
| **Hash Map** | `Map<Character, Node>` for character transitions in Trie | `AhoCorasick.java` |
| **JSON Objects** | Structured schema representation for file import/export | `sample_questions_*.json` |

---

## 4. Top 12 Viva / Review Questions & Model Answers

### Q1: Why did you choose this project for DSA-3?
**Answer:**  
"DSA-3 focuses on advanced pattern matching, tree structures, and dynamic programming. A Question Bank Management System directly requires:
1. Linear string matching (KMP and Rabin-Karp) for instant question discovery.
2. Trie-based multi-pattern matching (Aho-Corasick) for syllabus keyword mapping.
3. Dynamic Programming (Levenshtein Distance) for typo tolerance and duplicate detection."

### Q2: What is the main drawback of Naive pattern search that KMP overcomes?
**Answer:**  
"The Naive search has a worst-case time of $O(N \times M)$ because it backtracks the text index $i$ to the next starting character upon mismatch. KMP computes the LPS table beforehand in $O(M)$ time and never backtracks the text pointer $i$, strictly achieving $O(N + M)$ time."

### Q3: What is a spurious hit in Rabin-Karp, and how do you resolve it?
**Answer:**  
"A spurious hit is a hash collision where the sliding window in the text produces the same numerical hash value as the pattern, but the actual characters differ. When the hashes match, Rabin-Karp resolves this by executing a character-by-character check."

### Q4: How does rolling hash achieve $O(1)$ window transitions?
**Answer:**  
"Instead of recalculating the hash from scratch in $O(M)$ time, rolling hash removes the leading character's value $(text[i] \times d^{M-1})$ and adds the new incoming character $(text[i+M])$ using modular arithmetic in $O(1)$ constant time."

### Q5: When should Aho-Corasick be preferred over KMP?
**Answer:**  
"KMP is optimized for finding a single pattern. If an educator wants to search for multiple syllabus topics simultaneously (e.g., 5 to 10 keywords), running KMP 10 times costs $O(K \times (N+M))$. Aho-Corasick searches all patterns simultaneously in a single text traversal in $O(N + M_{\text{total}} + Z)$ time."

### Q6: How are failure links in Aho-Corasick constructed?
**Answer:**  
"Failure links are built using Breadth-First Search (BFS) level-by-level starting from the root node. For any node representing string $P \cdot c$, its failure link points to the longest proper suffix of $P \cdot c$ that exists as a valid prefix in the Trie."

### Q7: How does your system detect duplicate or rephrased questions?
**Answer:**  
"We calculate the Levenshtein Distance between questions using dynamic programming. We then normalize the distance by the maximum string length to derive a percentage similarity score. If the similarity is $75\%$ or higher, our system alerts the educator."

### Q8: What are the operations in Levenshtein Distance?
**Answer:**  
"Three operations:
1. Insertion (cost 1)
2. Deletion (cost 1)
3. Substitution (cost 1 if characters differ, cost 0 if identical)."

### Q9: Can the space complexity of Levenshtein Distance be optimized?
**Answer:**  
"Yes. In the 2D DP matrix, each cell $dp[i][j]$ depends only on the current row and the previous row ($dp[i-1]$). Hence, space can be reduced from $O(N \times M)$ to $O(\min(N, M))$ using just two 1D arrays."

### Q10: How does role-based authentication work in QBMS?
**Answer:**  
"We provide dual roles:
- **Student Role:** Can view, filter, practice, and test pattern matching searches.
- **Admin/Faculty Role:** Full privileges including adding, modifying, and deleting questions, running duplicate detection, and generating randomized examination papers."

### Q11: How does the Automated Question Paper Generator function?
**Answer:**  
"The faculty configures total marks, subject, distribution across Units (1 to 5), and difficulty weights (e.g., 40% Easy, 40% Medium, 20% Hard). The system filters the matching questions from the repository and uses randomized selection without replacement to construct a balanced examination paper that can be exported to PDF or JSON."

### Q12: What was your specific individual contribution (D. Mohit Venkat Sai)?
**Answer:**  
"I focused on core algorithm design and dynamic programming—specifically implementing the Knuth-Morris-Pratt (KMP) exact matcher and the Levenshtein Distance dynamic programming matrix for fuzzy search and duplicate detection. I also contributed to testing the algorithms and developing the interactive visualization layer."

---

## 5. Live Demo Execution Plan

1. **Terminal (Java CLI):**
   ```bash
   javac Main.java
   java Main
   ```
   - Select Option `2`: Display sample questions.
   - Select Option `5`: Demonstrate KMP search for `"Process"`.
   - Select Option `10`: Demonstrate Duplicate Detection on similar questions.

2. **Web Portal (`index.html`):**
   - Open `index.html` directly in Google Chrome / Edge.
   - Click quick sample card **`EMP1001` (Prof. Suresh Sharma)** to log in as Admin.
   - Navigate to **Algorithm Visualizer** tab:
     - Select **KMP** $\rightarrow$ Show the LPS table construction and index tracking.
     - Select **Levenshtein** $\rightarrow$ Show the 2D DP matrix computation.
   - Navigate to **Paper Generator** $\rightarrow$ Click **Generate Paper** $\rightarrow$ Show the balanced question paper preview.
