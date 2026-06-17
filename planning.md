# FitFindr — planning.md

> Complete this document before writing any implementation code.
> Your spec and agent diagram are what you'll use to direct AI tools (Claude, Copilot, etc.) to generate your implementation — the more specific they are, the more useful the generated code will be.
> Your planning.md will be reviewed as part of your submission.
> Update it before starting any stretch features.

---

## Tools

List every tool your agent will use. For each tool, fill in all four fields.
You must have at least 3 tools. The three required tools are listed — add any additional tools below them.

### Tool 1: search_listings

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
This tool filters the mock listings dataset to find secondhand clothing items that match the user's specific criteria for style, size, and budget.

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `description` (str): ...A natural language string describing the item (e.g., "vintage graphic tee").
- `size` (str): ...The desired size (e.g., "M", "L", "XL").
- `max_price` (float): ... The maximum budget for the item.

**What it returns:**
<!-- Describe the return value — what fields does a result contain? -->
A list of dictionaries, where each dictionary represents a matching listing with fields including id, title, description, brand, price, size, condition, colors, and platform

**What happens if it fails or returns nothing:**
<!-- What should the agent do if no listings match? -->
The tool returns an empty list []. The agent must then inform the user that no matches were found, suggest they loosen their constraints (like increasing the price or changing the description), and terminate the workflow immediately

---

### Tool 2: suggest_outfit

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
It takes a specific clothing item and the user's existing wardrobe to generate one or more creative outfit combinations with specific styling tips.

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `new_item` (dict): ...It takes a specific clothing item and the user's existing wardrobe to generate one or more creative outfit combinations with specific styling tips.
- `wardrobe` (dict): ...A dictionary containing a list of items currently owned by the user.

**What it returns:**
<!-- Describe the return value -->
A descriptive string containing a full outfit combination and advice on how to wear it (e.g., "Pair this with your wide-leg jeans and platform Docs for a classic 90s grunge look")

**What happens if it fails or returns nothing:**
<!-- What should the agent do if the wardrobe is empty or no outfit can be suggested? -->
A descriptive string containing a full outfit combination and advice on how to wear it (e.g., "Pair this with your wide-leg jeans and platform Docs for a classic 90s grunge look")

---

### Tool 3: create_fit_card

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
 It generates a short, creative, social-media-style caption (e.g., for Instagram) for the final suggested outfit.
 
**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `outfit` (str): ...The styling suggestion string produced by suggest_outfit.
- `new_item` (dict): ...The dictionary of the selected item.

**What it returns:**
<!-- Describe the return value -->
A creative caption string that incorporates the item details and styling context, producing unique results for different inputs


**What happens if it fails or returns nothing:**
<!-- What should the agent do if the outfit data is incomplete? -->
 If the outfit data is incomplete or empty, the tool returns a descriptive error message string (e.g., "Could not generate fit card due to missing outfit details") rather than a Python exception

---

### Additional Tools (if any)

<!-- Copy the block above for any tools beyond the required three -->

---

## Planning Loop

**How does your agent decide which tool to call next?**
<!-- Describe the logic your planning loop uses. What does it look at? What conditions change its behavior? How does it know when it's done? -->

The planning loop follows a conditional logic path rather than a fixed sequence. It starts by calling search_listings. It then checks the result: if the list is empty, it sets an error message in the session state and returns to the user immediately. If items are found, it selects the top result and proceeds to suggest_outfit. The loop is complete once create_fit_card has successfully generated a caption or an error branch is triggered
.

---

## State Management

**How does information from one tool get passed to the next?**
<!-- Describe how your agent stores and accesses state within a session. What data is tracked? How is it passed between tool calls? -->

Information is stored in a session dictionary that persists across the interaction. The agent tracks session["selected_item"] from the search tool, session["outfit_suggestion"] from the styling tool, and session["fit_card"] from the caption tool. Each subsequent tool accesses these specific keys from the session state to perform its task without requiring the user to re-input data
---

## Error Handling

For each tool, describe the specific failure mode you're handling and what the agent does in response.

| Tool | Failure mode | Agent response |
|------|-------------|----------------|
| search_listings | No results match the query | |
| suggest_outfit | Wardrobe is empty | |
| create_fit_card | Outfit input is missing or incomplete | |


---

## Architecture

<!-- Draw a diagram of your agent showing how the components connect:
     User input → Planning Loop → Tools (search_listings, suggest_outfit, create_fit_card)
                                                                          ↕
                                                                   State / Session
     Show what triggers each tool, how state flows between them, and where error paths branch off.
     Use ASCII art or a Mermaid diagram (https://mermaid.js.org/syntax/flowchart.html).
     Do NOT embed an image — graders need to read your diagram directly in the file;
     an embedded image or screenshot cannot be evaluated.
     You'll share this diagram with an AI tool when asking it to implement
     the planning loop and each individual tool. -->
     
    A[User Query] --> B{Planning Loop}
    B --> C[search_listings]
    C -->|Empty []| D[ERROR: Notify User & Stop]
    C -->|Matches Found| E[Session: Store selected_item]
    E --> F[suggest_outfit]
    F --> G[Session: Store outfit_suggestion]
    G --> H[create_fit_card]
    H --> I[Session: Store fit_card]
    I --> J[Return Session to User]
    D --> J


---

## AI Tool Plan

<!-- For each part of the implementation below, describe:
     - Which AI tool you plan to use (Claude, Copilot, ChatGPT, etc.)
     - What you'll give it as input (which sections of this planning.md, your agent diagram)
     - What you expect it to produce
     - How you'll verify the output matches your spec before moving on

     "I'll use AI to help me code" is not a plan.
     "I'll give Claude my Tool 1 spec (inputs, return value, failure mode) and ask it to implement
     search_listings() using load_listings() from the data loader — then test it against 3 queries
     before trusting it" is a plan. -->

**Milestone 3 — Individual tool implementations:**

**Milestone 4 — Planning loop and state management:**

---


## A Complete Interaction (Step by Step)

**Example user query:** "I'm looking for a vintage graphic tee under $30. I mostly wear baggy jeans and chunky sneakers. What's out there and how would I style it?"

**Step 1:**
The agent parses the user's natural language request to extract parameters for the **`search_listings`** tool. It calls the tool with the following inputs: `description="vintage graphic tee"`, `size=None` (as no size was specified), and `max_price=30.0`. The tool searches the mock dataset and returns a list of matching items. If no matches are found, the agent stops here and informs the user; otherwise, it selects the top result (e.g., "Faded Band Tee — $22").

**Step 2:**
The agent takes the selected item from Step 1 and the user's wardrobe data (retrieved via `get_example_wardrobe()`) and passes them into the **`suggest_outfit`** tool. This tool generates specific styling advice based on both the new item and what the user already owns. For example, it might return: *"Pair this with your baggy jeans and chunky sneakers for a relaxed 90s aesthetic. Try a partial tuck to highlight the waistline"*.

**Step 3:**
Finally, the agent passes the outfit suggestion from Step 2 and the item details from Step 1 into the **`create_fit_card`** tool. This tool generates a creative, shareable caption for the look. The agent ensures that the information flows seamlessly between these calls using **state management**, so the user does not have to re-enter any details.

**Final output to user:**
The user sees the found item, the detailed styling suggestion, and a ready-to-use social media caption, such as: *"thrifted this faded band tee for $22 and honestly it was made for my baggy jeans 🖤 full look in my stories"*.










