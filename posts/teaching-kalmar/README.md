# Teaching Kalmar's Construction Strategy

The [Spar Project](https://sparai.org/projects/f26/rec8qbsiN63iT3e8Z/)'s domain is propositional logic. This domain has a decidable algorithm to generate a proof (or counter-proof) of any theorem. One such algorithm is Kalmar's construction strategy, which was used to show that the domain is decidable.

In my own butchered terms, you enumerate the atoms (in our case [PQRS]) and do case analysis on each possible assignment of truth values. For us, that is at most 2^4 (16) cases. For each case, you either directly show that the theorem-to-be-proved is true, or you select a premise which is false in this case (and use ex falso to then conclude that the theorem-to-be-proved is 'still' true).

I had Claude write up [a short program](https://github.com/chainik1125/nd-rl/blob/dan/code/tools/tt_prover.py) to implement this algorithm. To no one's surprise, it correctly makes proofs in our small domain, and does so very quickly ([longer analysis](https://github.com/chainik1125/nd-rl/blob/dan/docs/project_strategy/kalmar-vs-model-efficiency/README.md)). The lengths of these proofs are exponential relative to the number of atoms considered, so extending our problems to 7-12 atoms lets the models we train have a fighting chance of beating that little script. The script does have the advantage of being immediately intelligible and guaranteed to find a proof or counter-proof though.

Out of morbid curiousity, I tried training the model to use this construction technique. The model gets a lot of the way there, but deterministically following a long procedure over hundreds of steps is not really the strong suite of neural networks. So it beats our default trained model on some classes of problems, but loses on many others (due to failing to follow the procedure with enough exactitude). Claude's analysis is [here](https://github.com/chainik1125/nd-rl/blob/dan/docs/project_strategy/kalmar-vs-model-efficiency/README.md).

I'm not totally sure what to draw from this. Maybe making me even less interested in improving our model's performance, relative to examining learning dynamics. Like, we're not gonna make a better solver, so I just want to understand how learning happens for the current models. Maybe if we train a better model we'd have better external validity since we're at least close to the training efficiency fronter? I don't think I really buy that tho.

So other folks might try to push model performance for the project, but I really don't find it compelling to try to make a 10M parameter model try to beat 200 lines of Python XD

## But how hard actually is Kalmar?
[680 bytes or less](https://github.com/chainik1125/nd-rl/blob/dan/experiment-summaries/2026-10-20-kalmar-complexity-5443-bit-program-per-step-demand-not-program-length-ranks-kal-d-errors/plain_summary.md), when placed on top of Python (which is of course cheating in many ways). The same analysis showed direct truth table checking was less than a tenth the size.

Over the course of training, the model nominally absorbed many more bits than the length of the program (Claude gives a range of 50-1300X). That makes sense to me intuitively, as training over 155k proofs is gonna be much more than 680 bytes of info.

But it isn't necessarily surprising that the generated proofs do not optimally convey the simple algorithm. I feel like you'd need to have a pretty fancy prior that puts weight on 'simple' algorithms in some sense before you caught on to this pattern from this info. But I'm far from certain of that, it could be that the training had enough data for the model to learn the algorithm & our training procedure was just flawed. Hard to think how to handle partially negative results like this. Its always possible that the answer is a skill issue thing on my part, rather than a fundamental limitation which is interesting & generalizable.

Claude also noted that certain steps are much more error prone than others, particularly searching a formula for the next part not-yet-proved. This makes sense, but honestly wasn't something I had already thought of. Interesting that the algorithm has easier & harder to learn pieces. Claude also trained a small model trying to improve performance on just these difficult steps, and did indeed get some improvements. Neat.

Claude also noted that the length of the generator code does not strictly correspond to the difficulty of learning the procedure, which makes sense but is also good to see cross-checked.
