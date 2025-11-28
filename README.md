#+OPTIONS: toc:nil

* Overview

SynthSMV (supervisor SYNTHesis using the new Symbolic Model Verifier)
is an extension of the model checking tool NuSMV (version 2.6.0) to
solve a certain class of supervisory control problems.

Given:

1. A discrete event system modeled in NuSMV (which corresponds to a
   deterministic finite labeled transition system).

2. A set of controllable events.

3. A computation tree logic (CTL) specification of the form
   ~$AG(\bigwedge\limits_{i \in I}p_i \wedge \bigwedge\limits_{j \in
   J}EF(p_j))$~, where the ~$p_i$~ and ~$p_j$~ do not contain any
   additional temporal operators.

SynthSMV computes the maximally-permissive state-based supervisor (a
map from the current state of the system to the set of enabled events)
that enforces the specification.

SynthSMV extends the input language of NuSMV with two new keywords:

1. ~CTRBL~ is used to specify which of the events (state/input pairs)
   are /controllable/.

2. ~SYNTH~ is used to define specifications (in a restricted subset of
   CTL, described earlier) for which a supervisor should be
   /synthesized/.

SynthSMV uses the BDD-based model checking algorithms in NuSMV.  The
SAT-based bounded model checking features of NuSMV are not used (but
were not removed from SynthSMV).

For more information, to download the latest version of SynthSMV, or
to get in touch with the authors, please visit
[[https://bitbucket.org/blakecraw/synthsmv]].

* Example

If you are familiar with NuSMV and Supervisory Control Theory, the
following simple example should demonstrate the type of problems that
SynthSMV can solve:

#+BEGIN_SRC nusmv
  MODULE main

  VAR state : {1, 2, 3, 4};
  IVAR event : {1, 2};

  INIT state = 1;

  ASSIGN next(state) :=
    case
      state = 1:
        case
          event = 1: 2;
          event = 2: 4;
        esac;
      state = 2:
        case
          event = 1: 2;
          event = 2: 3;
        esac;
      state = 3:
        case
          event = 1: 2;
          event = 2: 4;
        esac;
      state = 4: 4;
    esac;

  CTRBL event = 2;

  SPEC AG(EF(state = 2) & EF(state = 3)); -- false

  SYNTH AG(EF(state = 2) & EF(state = 3)); -- true (with the appropriate
                                           -- control action)
#+END_SRC

The ~SPEC~ specification applies regular CTL model checking to the
uncontrolled system; the specification is ~false~, because the system
can reach state ~4~, at which point states ~2~ and ~3~ are no longer
reachable.  The ~SYNTH~ specification applies supervisor synthesis to
enforce the specification; the specification is ~true~ in the
controlled system, with the supervisor preventing any events that
cause a transition to state ~4~.

* Expected Output

When the ~-v 1~ command-line argument is passed, SynthSMV reports some
information about the supervisor.

SynthSMV displays the supervisor that it computed as the formula that
represents the new state/input constraints that were applied to the
controlled system.  This formula can be applied directly to the
uncontrolled system in the form of a single ~TRANS~ constraint in the
model.  In the example shown earlier, SynthSMV produces the
supervisor:

#+BEGIN_SRC nusmv
  case
  2 = event : (2 = state | 4 = state);
  TRUE : 1 = event;
  esac
#+END_SRC

which can be applied to the original model:

#+BEGIN_SRC nusmv
  MODULE main

  VAR state : {1, 2, 3, 4};
  IVAR event : {1, 2};

  INIT state = 1;

  ASSIGN next(state) :=
    case
      state = 1:
        case
          event = 1: 2;
          event = 2: 4;
        esac;
      state = 2:
        case
          event = 1: 2;
          event = 2: 3;
        esac;
      state = 3:
        case
          event = 1: 2;
          event = 2: 4;
        esac;
      state = 4: 4;
    esac;

  CTRBL event = 2;

  -- The supervisor, as a single `TRANS` constraint.
  TRANS
  case
  2 = event : (2 = state | 4 = state);
  TRUE : 1 = event;
  esac
  ;

  SPEC AG(EF(state = 2) & EF(state = 3)); -- true

  SYNTH AG(EF(state = 2) & EF(state = 3)); -- true (with no additional
                                           -- control action required)
#+END_SRC

so that the ~SPEC~ specification is ~true~ in the new model, and the
~SYNTH~ specification is also ~true~, and does not produce any
additional constraints, as expected.

SynthSMV also shows some information about the number of iterations
required when computing the supervisor, the number of BDD nodes
required to represent the supervisor, and the reachable states in the
uncontrolled and controlled system, to give an idea of what action the
supervisor is taking to enforce a specification.

If the problem cannot be solved (i.e., the specification is ~false~
even in the controlled system), SynthSMV produces a counterexample
trace using its "best attempt" at a working supervisor.

* Building, Installation, and Supported Platforms

Please refer to the files =NuSMV-modified/README_NuSMV.txt= and
=NuSMV-modified/README_PLATFORMS_NuSMV.txt=, which have detailed
information and instructions (these are the instructions that come
with NuSMV, and should also work for SynthSMV).

* Useful Links

** NuSMV

[[http://nusmv.fbk.eu/]]

** CUDD

[[http://vlsi.colorado.edu/~fabio/CUDD/]]

* Copyright and License Information

Copyright (c) 2015-2016, Blake C. Rawlings.

SynthSMV is free software; you can redistribute it and/or modify it
under the terms of the GNU Lesser General Public License as published
by the Free Software Foundation; either version 2.1 of the License, or
(at your option) any later version.

SynthSMV is distributed in the hope that it will be useful, but
WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
Lesser General Public License for more details.

You should have received a copy of the GNU Lesser General Public
License along with SynthSMV.  If not, see
[[http://www.gnu.org/licenses/]].

** NuSMV (version 2.6.0)

NuSMV version 2 (NuSMV 2 in short) is licensed under the GNU Lesser
General Public License (LGPL in short). File LGPL-2.1 contains a copy
of the License.

The aim of the NuSMV OpenSource project is to allow the whole model
checking community to participate to the development of NuSMV. To this
purpose, we have chosen a license that:
1. is "copyleft", that is, it requires that anyone who improves the
   system has to make the improvements freely available;
2. permits to use the system in research and commercial
   applications, without restrictions.

In brief, the LGPL license allows anyone to freely download, copy,
use, modify, and redistribute NuSMV 2, proviso that any modification
and/or extension to the library is made publicly available under the
terms of LGPL.

The license also allows the usage of the NuSMV 2 as part of a larger
software system *without* being obliged to distributing the whole
software under LGPL. Also in this case, the modification to NuSMV 2
(*not* to the larger software) should be made available under LGPL.

The precise terms and conditions for copying, distribution and
modification can be found in file LGPL-2.1. You can contact
<nusmv@fbk.eu> if you have any doubt or comment on the license.

Different partners have participated the initial release of
NuSMV 2. Every source file in the NuSMV 2 distribution contains a
header that acknowledges the developers and the copyright holders for
the file. In particular:

- CMU and ITC-IRST contributed the source code on NuSMV version 1.
- ITC-IRST has also developed several extensions for NuSMV 2.
- ITC-IRST and the University of Trento have developed the
  SAT-based Bounded Model Checking package on NuSMV 2.
- the University of Genova has contributed SIM, a state-of-the-art
  SAT solver used until version 2.5.0, and the RBC package use in the
  Bounded Model Checking algorithms.
- Fondazione Bruno Kessler (FBK) is currenlty the main developer
  and maintainer of NuSMV 2.

The NuSMV team has also received several contributions for different
part of the system. In particular:

- Ariel Fuxman <afuxman@cs.toronto.edu> has extended the LTL to SMV
  tableau translator to the past fragment of LTL
- Rik Eshuis <eshuis@cs.utwente.nl> has contributed a strong
  fairness model checking algorithm for LTL specifications
- Dan Sheridan <dan.sheridan@contact.org.uk> has contributed
  several extensions and enhancements to the Bounded Model Checking
  algorithms.

** CUDD (version 2.4.1)

Copyright (c) 1995-2004, Regents of the University of Colorado

All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are
met:

Redistributions of source code must retain the above copyright notice,
this list of conditions and the following disclaimer.

Redistributions in binary form must reproduce the above copyright
notice, this list of conditions and the following disclaimer in the
documentation and/or other materials provided with the distribution.

Neither the name of the University of Colorado nor the names of its
contributors may be used to endorse or promote products derived from
this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
"AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

** MiniSat (release 2.2)

MiniSat -- Copyright (c) 2003-2006, Niklas Een, Niklas Sorensson
           Copyright (c) 2007-2010 Niklas Sorensson

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

** ZChaff64 (2007.03.12; linked optionally)

Copyright 2000-2004, Princeton University.  All rights reserved.  By
using this software the USER indicates that he or she has read,
understood and will comply with the following:

--- Princeton University hereby grants USER nonexclusive permission to
use, copy and/or modify this software for internal, noncommercial,
research purposes only. Any distribution, including commercial sale or
license, of this software, copies of the software, its associated
documentation and/or modifications of either is strictly prohibited
without the prior consent of Princeton University.  Title to copyright
to this software and its associated documentation shall at all times
remain with Princeton University.  Appropriate copyright notice shall
be placed on all software copies, and a complete copy of this notice
shall be included in all copies of the associated documentation.  No
right is granted to use in advertising, publicity or otherwise any
trademark, service mark, or the name of Princeton University.


--- This software and any associated documentation is provided "as is"

PRINCETON UNIVERSITY MAKES NO REPRESENTATIONS OR WARRANTIES, EXPRESS
OR IMPLIED, INCLUDING THOSE OF MERCHANTABILITY OR FITNESS FOR A
PARTICULAR PURPOSE, OR THAT USE OF THE SOFTWARE, MODIFICATIONS, OR
ASSOCIATED DOCUMENTATION WILL NOT INFRINGE ANY PATENTS, COPYRIGHTS,
TRADEMARKS OR OTHER INTELLECTUAL PROPERTY RIGHTS OF A THIRD PARTY.

Princeton University shall not be liable under any circumstances for
any direct, indirect, special, incidental, or consequential damages
with respect to any claim by USER or any third party on account of or
arising from the use, or inability to use, this software or its
associated documentation, even if Princeton University has been
advised of the possibility of those damages.
