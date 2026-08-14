# The Second Loop: What AI Does to Your Opinion of Yourself

We have spent the better part of a decade asking whether people trust AI appropriately. Do they rely on it too much? Too little? Does their reliance track what the system can actually do? That question matters, and it is the one I have staked my own research on. It sits squarely in the tradition that Lee and See (2004) established: trust should be calibrated to the true capabilities of the automation. But a new paper in the *Journal of Intelligence* makes a case that we have been watching only half the problem.

Monica Maier's "Self-Evaluation in AI-Assisted Cognition" (Maier, 2026) is a conceptual review, not an experiment, and I usually approach conceptual reviews with my hand on my wallet. This one earns its keep. It pulls together research on metacognition, self-regulated learning, cognitive offloading, and trust in AI, and it names a gap that anyone building or governing AI-assisted work should sit with: nobody has a good account of how AI-assisted success becomes evidence, in a person's own mind, of their own competence.

Read that again, because it is sneaky. The question is not whether AI makes you better at the task. It often does. The question is what the improved result does to your judgment of yourself.

## Two loops, not one

Here is the frame I took away from the paper, translated into the language I use in my own work.

When you work with an AI system, there are two calibration loops running.

The first loop is the one the human-machine teaming community knows well: your model of the machine. Is this system reliable for this task? Should I accept this output or check it? This is trust calibration in the Lee and See (2004) tradition, and it is where most of the instrumentation, research, and design attention has gone.

The second loop is your model of yourself. After the task is done, what do you believe about your own competence? Could you do this again without the assistant? Do you understand the result well enough to defend it, transfer it, rebuild it?

The uncomfortable finding threaded through Maier's review is that these loops can move in opposite directions. Fernandes and colleagues put it memorably in the title of their study: AI makes you smarter but none the wiser (Fernandes et al., 2026). In their first experiment, participants using ChatGPT on logical reasoning problems drawn from the LSAT performed about three points better than a norm population, and simultaneously overestimated their own performance by about four points. A second study replicated the pattern. Two details deserve their own paragraph.

First, the Dunning-Kruger effect, the familiar finding that the weakest performers overestimate themselves the most, disappeared under AI use. Everyone overestimated, across the ability range. Second, and this is the one that should sting for readers of this blog, higher AI literacy was associated with worse metacognitive accuracy, not better (Fernandes et al., 2026). Knowing more about the tools made people more confident and less precise about their own performance. If your defense against overtrust is "our people are sophisticated users," this result is aimed directly at you.

The feedback story is no more comforting. Liebenow and colleagues ran a randomized controlled experiment on LLM-generated feedback and found it did not improve self-assessment accuracy on average, though it did help students whose self-assessment was poorly calibrated to begin with (Liebenow et al., 2025). That differential effect is worth holding onto: AI feedback can be a corrective for the worst-calibrated, and inert for everyone else. The OECD (2026) has started calling the aggregate version of this pattern a "mirage of false mastery."

So you can be perfectly calibrated on the first loop and drifting badly on the second. You correctly trust a capable system, the output is good, the work ships, and somewhere in that process the success quietly migrates onto your own ledger. The system's contribution was real but opaque, and opacity plus fluency is exactly the recipe for what Maier (2026) calls erroneous attribution of success.

## Why the fluency is the trap

Maier organizes her framework around eight axes, and I will not walk through all of them here. But two distinctions do most of the work, and they are worth carrying around.

The first is where AI enters the task (Maier, 2026, Axis 1). AI that shows up as a resource for comparison, critique, and revision keeps you connected to the process that produced the result. AI that shows up as a generator of finished output compresses that process into a black box. You still get a product. You just lose access to the path, and the path is where the cues for honest self-assessment live.

The second is what happens to the effort you saved (Maier, 2026, Axis 4). Offloading is not the villain. Pilots offload to autopilot, and good ones reinvest the freed attention into monitoring the bigger picture. The failure mode is when saved effort simply exits the task. You did less, verified less, compared less, and the polished result in front of you gives you no signal that anything is missing. The product looks the same either way. Your competence does not.

## The training analogy I cannot unsee

I coach hybrid athletes, so forgive me, but this is an assisted rep problem.

If you only ever squat with a spotter taking twenty percent of the load, two things are true at once. Your training numbers go up, and your actual max becomes unknown to you. The bar path feels smooth, the sessions look great in the log, and the day you have to move the weight alone is the day you discover what was yours and what was the spotter's. No serious athlete confuses assisted volume with tested strength. We test unassisted on purpose, at intervals, because the gap between the two numbers is the single most important thing to know about your training.

Knowledge work with AI has no such convention yet. We are logging assisted volume every day and almost never testing the unassisted max. Yan and colleagues made this point sharply for education: generative AI can produce performance gains without the deep processing that constitutes actual learning, and the two are easy to confuse precisely because the visible artifact looks the same (Yan et al., 2025). Maier's sixth axis, the gap between assisted performance and actual learning, formalizes this, and her point is that self-evaluation becomes vulnerable in proportion to the size of that gap (Maier, 2026). You start estimating your one-rep max from your spotted sets.

## What to actually do about it

Because this is a conceptual paper, the practical guidance stays at the level of design principles. But the principles are good, and they translate directly into things teams and organizations can implement now.

**Make the contribution split visible.** The single highest-leverage move is provenance: knowing, at the level of the finished product, what was generated, what was revised, what was human-authored, and what was verified. Not for surveillance. For honesty. Two identical documents with different provenance are not the same document, and the author deserves to see the difference as much as the reviewer does. Maier (2026) identifies opacity of the AI contribution as a core driver of erroneous attribution, and provenance is opacity's direct antidote.

**Commit before you consult.** Maier's strongest workflow recommendation is independent-first: form your own position, even a rough one, before the assistant weighs in (Maier, 2026). The initial commitment gives you a before-and-after view of your own thinking, which is the raw material of calibration. Teams can make this a norm. Tooling can make it a default.

**Ask attribution questions, not confidence questions.** "How confident are you in this answer?" conflates the two loops. Better questions, adapted from the self-evaluation instruments Maier proposes: Which parts of this could you defend without the system? What did you independently verify? What would you have concluded on your own? These are cheap to ask, and the Fernandes team reached a similar design conclusion, suggesting that AI systems should prompt users to explain their reasoning in order to confront the illusion of knowledge (Fernandes et al., 2026).

**Schedule the unassisted test.** Periodically, deliberately, do the task cold. Not as a purity ritual and not as a performance review, but as a measurement. If the gap between your assisted and unassisted work is growing, that is information you want early, while it is still a training problem and not an operational one. In some domains, the ability to perform when the tooling is degraded is not a pedagogical nicety. It is continuity of operations.

## The part that stays with me

The trust calibration field, my field, has been asking whether humans trust machines correctly. Maier's paper is a reminder that the machine was never the only thing in the room being evaluated. Every AI-assisted task is also, quietly, an update to a person's beliefs about themselves, and right now those updates are running on corrupted evidence: fluent products whose provenance the person can no longer fully reconstruct.

The systems we should be building do not just earn appropriate trust. They keep the human's contribution legible enough that the human's self-trust stays honest too. That is a design requirement, not a wellness suggestion, and it belongs in the same conversation as accuracy, safety, and reliability.

Both loops. Or the performance gains are writing checks the competence cannot cash.

---

## References

Fernandes, D., Villa, S., Nicholls, S., Haavisto, O., Buschek, D., Schmidt, A., Kosch, T., Shen, C., & Welsch, R. (2026). AI makes you smarter but none the wiser: The disconnect between performance and metacognition. *Computers in Human Behavior, 175*, 108779. https://doi.org/10.1016/j.chb.2025.108779

Lee, J. D., & See, K. A. (2004). Trust in automation: Designing for appropriate reliance. *Human Factors, 46*(1), 50-80. https://doi.org/10.1518/hfes.46.1.50_30392

Liebenow, L. W., Schmidt, F. T. C., Meyer, J., & Fleckenstein, J. (2025). Self-assessment accuracy in the age of artificial intelligence: Differential effects of LLM-generated feedback. *Computers & Education, 237*, 105385. https://doi.org/10.1016/j.compedu.2025.105385

Maier, M. (2026). Self-evaluation in AI-assisted cognition: An explanatory framework for calibration and miscalibration effects. *Journal of Intelligence, 14*(7), 112. https://doi.org/10.3390/jintelligence14070112

OECD. (2026). *OECD digital education outlook 2026: Exploring effective uses of generative AI in education.* OECD Publishing.

Yan, L., Greiff, S., Lodge, J. M., & Gasevic, D. (2025). Distinguishing performance gains from learning when using generative AI. *Nature Reviews Psychology, 4*, 435-436.
