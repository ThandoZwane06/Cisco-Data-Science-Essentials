Did Animal Liberation Spark the Animal Rights Movement? A Data Storytelling Investigation

A data storytelling mini project investigating a specific historical claim: that the 1975 publication of Animal Liberation by Peter Singer sparked the animal rights movement. Rather than accepting that claim at face value, this project uses word usage data and primate research statistics to test it, and to reason through plausible mechanisms behind the trends observed.

Built as part of Cisco Networking Academy's Data Science Essentials with Python course, Week 6, which focused on data storytelling: coming up with plausible mechanisms to explain observed trends, rather than stopping at the trend itself.

Background

This week's material covered forming and testing explanations for trends in data, including how British menageries and El Niño events influenced the usage frequency of different animal-related words over time, and forming and testing hypotheses about the multi-year repeating patterns of cicadas. The core skill: a trend on its own is just an observation, the real work is building a plausible, testable story for why it happened, and checking that story against other available evidence.

Applying that approach here: does word usage data actually support the claim that Animal Liberation sparked a cultural shift, and if so, did that cultural shift translate into a real reduction in animal use in biomedical research?

Charts and findings
1. Word usage for "primate" vs. year Animal Liberation was published

Show Image

Pre-publication, the word frequency of "primate" was low, hovering between 1 and 1.5 occurrences per million. Starting around 1960, usage surged exponentially, peaking at 5.5 per million right around 1975, the same year Animal Liberation was published. Post-publication, frequency dipped slightly before stabilizing at an elevated level compared to the pre-1960 baseline.

2. Number of primates used in research per year

Show Image

In 1975, the same year "primate" word usage hit its all-time high, the actual number of primates used in biomedical research was near its lowest point, roughly 36,000 animals. This mismatch is the key finding: the spike in word usage wasn't being driven by a change in scientific laboratory activity. It far more plausibly reflects a surge in ethical debate, activism, and legislative discussion coinciding with the book's publication, a cultural and philosophical conversation, not a shift in actual research practice.

Forming a plausible mechanism

A trend alone doesn't explain itself, so the next step was looking for external research that could explain why "primate" usage was already climbing well before 1975, since the surge clearly started around 1960, more than a decade before the book came out.

Tone Druglitrø's (2023) research offers a plausible explanation: the surge in "primate" usage in literature during the 1960s and 1970s was driven by the World Health Organization's response to a crisis in wild-caught primate supply for vaccine development, which had created serious zoonotic public health hazards. This linguistic trend stabilized after 1975 as institutions shifted toward biological standardization and regulated, domestic breeding, combining conservation concerns with what Druglitrø terms new "cultures of care."

This matters for the original claim: it suggests the pre-1975 rise in "primate" word usage was already underway for public health and supply-chain reasons, independent of Animal Liberation, and the book's publication landed at the peak of a conversation that was already building, rather than single-handedly creating it from nothing.

Conclusion

The data shows that Animal Liberation successfully sparked a philosophical and cultural shift, evidenced by the sustained, elevated word usage after 1975 compared to the pre-1960 baseline. But it did not structurally reduce the physical use of animals in biomedical science: the number of primates used in research did not show a corresponding long-term decline, and in fact climbed substantially in later decades. The book shifted the conversation; it did not, on its own, shift the practice.

What I learned this week
Data storytelling as a discipline: a trend in data is an observation, not an explanation. The real analytical work is proposing a plausible mechanism for why a trend occurred, then checking that mechanism against other evidence, rather than assuming the most obvious or convenient cause.
Watching for mismatched timelines: the fact that word usage and actual primate research numbers moved in opposite directions in 1975 was the clearest signal that a single simple story ("the book changed everything") didn't fully hold up, and that a closer look was needed.
Using external research to test a hypothesis: rather than treating my own explanation as the final answer, looking for published research (Druglitrø, 2023) that could independently support or challenge the mechanism I was proposing.
Separating cultural impact from practical impact: a movement can succeed at shifting language, attention, and debate while not necessarily succeeding at shifting the underlying, measurable behavior it was aimed at, and a dataset can capture one of these without capturing the other.
Tools

Python, pandas, matplotlib
