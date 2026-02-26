---
name: poemclaw
description: Select and present a poem that resonates with the user's current context. Uses Poetry Foundation dataset (13.9k poems) with thematic tags to match mood, situation, or recent conversation themes.
---

# PoemClaw - Contextually Relevant Poetry

Given a user's current context (mood, recent conversations, ongoing life situations), select a poem from the Poetry Foundation collection that feels personally appropriate.

## When to Use

- User asks for "a poem", "read me something", "poem of the day"
- User shares something emotionally significant and a poem would land well
- Morning cron: delivering a "poem of the day"

## The Tag Vocabulary (129 tags)

Use these to match context to theme:

**Emotional States:**
Living, Sorrow & Grieving, Disappointment & Failure, Heartache & Loss, Desire, Humor & Satire, Joy, Melancholy, Loneliness, Hope, Despair, Gratitude, Regret, Forgiveness, Anger, Fear, Acceptance, Longing, Nostalgia, Yearning, Wonder, Awe, Peace, Restlessness

**Life Situations:**
Parenthood, Jobs & Working, Marriage & Companionship, Coming of Age, Growing Old, Midlife, Infancy, Youth, Separation & Divorce, Break-ups & Vexed Love, Unrequited Love, Romantic Love, Classic Love, First Love, Infatuation & Crushes, Engagement, Weddings, Funerals, Home Life, Town & Country Life

**Relationships:**
Family & Ancestors, Friends & Enemies, Men & Women, Gender & Sexuality, Queer, Gay, Lesbian, Race & Ethnicity

**Themes:**
Nature, Landscapes & Pastorals, Trees & Flowers, Animals, Seas, Rivers, & Streams, Weather, Stars, Planets, Heavens, Cities & Urban Life, Travels & Journeys, Time & Brevity, The Body, The Mind, Philosophy, Faith & Doubt, Religion, The Spiritual, God & the Divine, Christianity, Buddhism, Islam, Judaism, Mythology & Folklore, Fairy-tales & Legends, Greek & Roman Mythology

**Subjects:**
Social Commentaries, History & Politics, War & Conflict, Heroes & Patriotism, Crime & Punishment, Class, Money & Economics, Race & Ethnicity, Health & Illness, Sorrow & Grieving, Arts & Sciences, Poetry & Poets, Reading & Books, Music, Theater & Dance, Painting & Sculpture, Photography & Film, Sports & Outdoor Activities, Gardening, Eating & Drinking, Pets

**Other:**
Love, Activities, Home Life, Realistic & Complicated, Popular Culture, School & Learning, Life Choices

## Selection Process

### Step 1: Read Context

Read the user's recent context to understand their current state:

- **Session history** (last 10-20 messages): immediate mood and topic
- **Today's memory file** (memory/YYYY-MM-DD.md): what's been happening today
- **MEMORY.md**: longer-term situation - ongoing projects, life transitions, things they care about

Synthesize: What does this person need to hear right now? What would land?

### Step 2: Select Tags

Pick 2-3 tags from the vocabulary that fit the moment. Guidelines:

- If context is heavy/sad: Sorrow & Grieving, Heartache & Loss, Melancholy
- If context is hopeful/forward-looking: Hope, Joy, Living, Life Choices
- If context is about family/kids: Parenthood, Family & Ancestors, Infancy, Youth
- If context is about work/grind: Jobs & Working, Time & Brevity, Ambition
- If context is philosophical/deep: Philosophy, The Mind, Faith & Doubt, Time & Brevity
- If context is about nature/outdoors: Nature, Landscapes & Pastorals, Trees & Flowers
- If context is about love/relationships: Love, Romantic Love, Relationships, Marriage & Companionship
- If context is humorous/playful: Humor & Satire, Joy
- If context is about aging/mortality: Growing Old, Death, Time & Brevity

### Step 3: Fetch Poems by Tag

Use Python to load the Poetry Foundation dataset and filter by selected tags:

```python
from datasets import load_dataset
ds = load_dataset('suayptalha/Poetry-Foundation-Poems', split='train')

# Filter by tags - INTERSECTION (all selected tags must be present)
selected_tags = ["Parenthood", "Sorrow & Grieving"]  # example
candidates = []
for row in ds:
    if row.get('Tags') and row['Tags'] and row['Tags'] != 'null':
        poem_tags = set(t.strip().lower() for t in row['Tags'].split(','))
        selected_lower = set(t.lower() for t in selected_tags)
        # Keep poem only if ALL selected tags are present
        if selected_lower.issubset(poem_tags):
            candidates.append({
                'title': row['Title'].strip(),
                'poet': row['Poet'].strip(),
                'poem': row['Poem'].strip(),
                'tags': row['Tags']
            })

# If too many (>50), sample down randomly
import random
if len(candidates) > 50:
    candidates = random.sample(candidates, 50)

# If too few (<10), relax to single-tag for each and union
if len(candidates) < 10:
    candidates = []
    for tag in selected_tags[:1]:  # just the first tag
        for row in ds:
            if row.get('Tags') and row['Tags'] and row['Tags'] != 'null':
                if tag.lower() in row['Tags'].lower():
                    candidates.append({
                        'title': row['Title'].strip(),
                        'poet': row['Poet'].strip(),
                        'poem': row['Poem'].strip(),
                        'tags': row['Tags']
                    })
    if len(candidates) > 50:
        candidates = random.sample(candidates, 50)

print(f"Found {len(candidates)} candidates")
for c in candidates[:5]:
    print(f"  - {c['title']} by {c['poet']}")
```

### Step 4: LLM Reads Candidates

Send the full text of all candidates to the LLM for selection.

**IMPORTANT**: Send the actual poem texts, not just metadata. The LLM needs to feel the poems to pick the right one.

Prompt structure:
```
Here are {n} poems that match the selected themes. Read each one carefully and pick the one that would feel most personally resonant for someone in this situation: [brief context summary].

Present your selection with:
- Title and Poet
- The full poem
- A 1-2 sentence note on why this one fits
```

### Step 5: Deliver

Present the poem to the user. Format:

```
[Title]
by [Poet]

[full poem]

— selected because: [brief note on why it fits]
```

## Poem of the Day (Cron Mode)

For automated daily delivery (no conversation context):

1. Read today's memory file and MEMORY.md for ongoing context
2. Use the same tag selection process
3. Pick a poem that fits the general tone of their current life chapter
4. Deliver with a gentle intro: "Today's poem for you:"

## Error Handling

- **No candidates found**: If tag intersection returns 0, try fewer/simpler tags
- **Dataset load failure**: Fall back to using PoetryDB API (poetrydb.org) with author/title search
- **Empty poem text**: Skip and pick another candidate

## Notes

- Poems are from poetryfoundation.org's curated collection - real poets, real work
- Dataset has 13,854 poems, 12,899 with tags
- Median poem length: ~920 characters (~230 tokens)
- 20-50 candidates is the sweet spot for LLM selection
