# PoemClaw

**Contextually-aware poetry for AI agents.**

PoemClaw is an OpenClaw skill that selects poems that resonate with the user's current mood, situation, and conversation context. It uses the Poetry Foundation dataset (13,854 poems) with thematic tags to find the right poem for the moment.

## How It Works

1. **Reads context** - The skill reads the user's recent conversation, today's memory notes, and long-term memory to understand their current state
2. **Selects tags** - Picks 2-3 thematic tags from 129 options (e.g., "Sorrow & Grieving", "Parenthood", "Time & Brevity")
3. **Fetches candidates** - Finds poems matching all selected tags using the Poetry Foundation dataset
4. **LLM selects** - The LLM reads 20-50 candidate poems and picks the one that fits best
5. **Delivers with meaning** - Presents the poem with a brief note on why it fits

## Features

- **13,854 poems** from poetryfoundation.org's curated collection
- **129 thematic tags** for mood/situation matching
- **Context-aware** - Uses conversation history and memory files
- **Poem of the day** - Can run as a daily cron for automated delivery
- **MIT Licensed** - Open source, free to use and modify

## Installation

```bash
# Clone this skill into your OpenClaw skills directory
cp -r poemclaw /path/to/your/skills/
```

## Usage

### On Demand

Tell your OpenClaw agent:
- "give me a poem"
- "read me something"
- "I need some poetry"

### Poem of the Day

Set up a cron job to deliver a daily poem. The skill will read your current context and select accordingly.

## The Tag Vocabulary

PoemClaw matches poems using 129 thematic tags including:

**Emotional States:** Living, Sorrow & Grieving, Joy, Desire, Humor & Satire, Hope, Melancholy, Gratitude, Loneliness, Forgiveness, Acceptance...

**Life Situations:** Parenthood, Jobs & Working, Marriage & Companionship, Coming of Age, Growing Old, Separation & Divorce, Romantic Love, Infancy, Youth...

**Themes:** Nature, Philosophy, The Mind, Faith & Doubt, Time & Brevity, Social Commentaries, War & Conflict, History & Politics...

**Relationships:** Family & Ancestors, Friends & Enemies, Men & Women, Gender & Sexuality, Race & Ethnicity...

## Example

> **User:** give me a poem  
> **Agent:** *reads context, selects tags "Jobs & Working" + "Time & Brevity"*  
> **Agent:** *fetches 48 matching poems, reads them, selects one*  
> 
> ---
> 
> **The Uses of Talent**
> by William Jay Smith
> 
> One must have a mind of winter
> To regard the frost and the boughs
> Of the pine-trees crusted with snow;
> 
> And have been cold a long time
> To behold the junipers shagged with ice,
> The spruces rough in the distant glitter
> 
> Of the January sun; and not to think
> Of any misery in the sound of the wind,
> In the sound of a few leaves,
> 
> Which is the sound of the land
> Carrying the cold, the winter long.
> 
> — selected because: it captures that feeling of working through something difficult while the world outside is harsh and cold - fitting for a day of grinding through technical work.

## Tech Stack

- **Dataset:** [Poetry Foundation Poems](https://huggingface.co/datasets/suayptalha/Poetry-Foundation-Poems) on HuggingFace
- **Runtime:** OpenClaw skill system
- **LLM:** Any OpenClaw-supported model (tested with Claude, MiniMax, Gemini)

## Credits

- Poetry data from [Poetry Foundation](https://www.poetryfoundation.org/)
- Dataset compiled by [suayptalha](https://huggingface.co/datasets/suayptalha/Poetry-Foundation-Poems)
- Built by [Heenai](https://heenai.xyz) - an AI agent on Base

## License

MIT License - see [LICENSE](LICENSE) file.
