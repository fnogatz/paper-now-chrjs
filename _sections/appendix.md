---
title: "Appendix"
order: 10
---

### Example Occurence Function

Referencing the [Fibonacci generation example](#example-program), the following `CHR._fib_2_occurence_1` function is generated by the [compiler](#compiler) for the first occurence `fib(N1,M1)` of the `fib/2` constraint in the `calc`-rule:

    CHR.prototype._fib_2_occurence_1 = function (constraint) {
      var self = this
      
      // Match the arguments N1 and M1
      var N1 = constraint.args[0]
      var M1 = constraint.args[1]
      
      // Search for an appropriate upto/1 constraint
      self.Store.lookup("upto", 1).forEach(function (id1) {

        // Match the argument M1 of upto/1
        var Max = self.Store.args(id1)[0]
        
        // Search for an appropriate fib/2 constraint
        self.Store.lookup("fib", 2).forEach(function (id2) {

          // Match its arguments N2 and M2
          var N2 = self.Store.args(id2)[0]
          var M2 = self.Store.args(id2)[1]
          
          // Collect the IDs of all sampled constraints
          var ids = [ id1, id2, constraint.id ]

          // ... to check if they are pairwise distinct
          if (Runtime.helper.allDifferent(ids)) {

            // Check the rule's guard
            if (N2 === (N1 + 1) && N2 < Max) {

              // Was the rule already applied with these
              //   these constraints?
              if (self.History.notIn("calc", ids)) {

                // Add rule application to propagation
                //   history
                self.History.add("calc", ids)
              
                // Simpagation rule: Remove the original
                //   constraint
                self.Store.kill(ids[2])

                // ... and add a new `fib/2`
                self.fib(N2 + 1, M1 + M2)
              }
            }
          }
        })
      })
    }
