University assignment for AI module - a constraint-satisfaction-based sudoku solver.

The main code can be found below and has been taken from the sudoku_csp .ipynb file.

```
# Lookup table for the 9 rows
rows = [[0,1,2,3,4,5,6,7,8],
        [9,10,11,12,13,14,15,16,17],
        [18,19,20,21,22,23,24,25,26],
        [27,28,29,30,31,32,33,34,35],
        [36,37,38,39,40,41,42,43,44],
        [45,46,47,48,49,50,51,52,53],
        [54,55,56,57,58,59,60,61,62],
        [63,64,65,66,67,68,69,70,71],
        [72,73,74,75,76,77,78,79,80]]

# Lookup table for the 9 columns
columns = [[0,9,18,27,36,45,54,63,72],
           [1,10,19,28,37,46,55,64,73],
           [2,11,20,29,38,47,56,65,74],
           [3,12,21,30,39,48,57,66,75],
           [4,13,22,31,40,49,58,67,76],
           [5,14,23,32,41,50,59,68,77],
           [6,15,24,33,42,51,60,69,78],
           [7,16,25,34,43,52,61,70,79],
           [8,17,26,35,44,53,62,71,80]]

# Lookup table for the 9 units
units = [[0,1,2,9,10,11,18,19,20],
        [3,4,5,12,13,14,21,22,23],
        [6,7,8,15,16,17,24,25,26],
        [27,28,29,36,37,38,45,46,47],
        [30,31,32,39,40,41,48,49,50],
        [33,34,35,42,43,44,51,52,53],
        [54,55,56,63,64,65,72,73,74],
        [57,58,59,66,67,68,75,76,77],
        [60,61,62,69,70,71,78,79,80]]

import math
import copy

class AlldiffConstraint():
    def __init__(self, positions):
        self.positions = positions

    def check_satisfied(self, assignment):
        """        If there are repeat values this method returns false (in python sets ignore duplicates)        """
        values = []
        for p in self.positions:
            if not assignment[p].value == 0:
                values.append(assignment[p].value)
        return len(values) == len(set(values))
#-------------------------------------------------------------------------------------------------------------------#

class Cell():
    def __init__(self, position, value):
        self.fixed = False
        self.prev_value = 0
        self.value = value
        self.position = position
        if not self.value == 0:
            self.domain = [value]
            self.fixed = True
        else:
            self.domain = [1,2,3,4,5,6,7,8,9]
        self.defDomain = set()
    
    def remove_from_domain(self, value):
        """ Returns true if the domain still has values in it, false if it's now empty """
        if not self.fixed:
            if self.domain.__contains__(value):
                self.domain.remove(value)
                #print("removed", value, "from domain of", self.position ,". Domain is now ", self.domain)
            if len(self.domain) == 0:
                #print("Domain of", self.position, "is now empty!")
                return False
            else:
                return True
        return True

    def add_to_domain(self, value):
        if not self.fixed:
            if not self.domain.__contains__(value):
                self.domain.add(value)
                #print("added", value, "to domain of", self.position, ". Domain is now ", self.domain)
        return True
    
    def update_value(self, new_value):
        
        self.prev_value = self.value
        self.value = new_value
        return True
        

    def restore_previous_value(self):
        #print("Restoring old value in position ", self.position, "- Old value: ", self.value, " | New value: ", self.prev_value)
        self.update_value(self.prev_value)

    def restore_domain(self):
        self.domain = self.defDomain.copy()
#---------------------------------------------------------------------------------------------------------------------#

def sudoku_solver(sudoku):
    """
    Solves a Sudoku puzzle and returns its unique solution.

    Input
        sudoku : 9x9 numpy array
            Empty cells are designated by 0.

    Output
        9x9 numpy array of integers
            It contains the solution, if there is one. If there is no solution, all array entries should be -1.
    """
    
    constraints = []
    # Row constraints
    for r in rows:
        constraints.append(AlldiffConstraint(r))
    # Column constraints
    for c in columns:
        constraints.append(AlldiffConstraint(c))
    # Unit constraints
    for u in units:
        constraints.append(AlldiffConstraint(u))

    assignment = []
    for row in range(0,9):
        for col in range(0,9):
            assignment.append(
                Cell(row*9+col, sudoku[row,col])
            )
            
    # Before starting the backtracking step, go through the assignment and update each non-fixed cell's domain
    unassigned = get_all_unassigned(assignment)
    for uc in unassigned:
        rowindex = math.floor((uc.position - (uc.position % 9)) / 9)
        colindex = (uc.position % 9)
        for u in units:
            if u.__contains__(uc.position):
                unitindex = units.index(u)
        
        existingValues = []
        for rc in rows[rowindex]:
            existingValues.append(assignment[rc].value)
        for cc in columns[colindex]:
            existingValues.append(assignment[cc].value)
        for uc1 in units[unitindex]:
            existingValues.append(assignment[uc1].value)

        defaultDomain = {1,2,3,4,5,6,7,8,9}
        uc.domain = defaultDomain.difference(set(existingValues))
        #print("Domain of unassigned in", uc.position ,"is now set as default:", uc.domain)
        for dv in uc.domain.copy():
            uc.defDomain.add(dv)
        if len(uc.domain) == 1:
            # There is only one value that can be here, so we can make it fixed
            uc.value = list(uc.domain)[0]
            uc.fixed = True

    result = backtrack(constraints, assignment)
    if result == []:
        return np.array([[ -1 for x in range(0, 9)] for y in range(0, 9)])
    else:
        # Reconstruct array from successful solve
        final_sudoku = np.zeros([9,9])
        i = 0
        for row in range(0,9):
            for col in range(0,9):
                final_sudoku[row,col] = result[i].value
                i+=1
        return final_sudoku
#----------------------------------------------------------------------------------------------------------------------#

def is_complete(constraints, assignment):
    """    Returns true if the supplied assignment is a valid, complete sudoku (satisfies all constraints)     """
    for constraint in constraints:
        if not constraint.check_satisfied(assignment):
            return False
        for cell in assignment:
            if cell.value == 0:
                return False
    return True

def get_all_unassigned(assignment):
    """    Returns a list of all the cells with unassigned values    """
    unassigned = []
    for cell in assignment:
        if cell.value == 0:
            unassigned.append(cell)
    return unassigned

def select_unassigned_variable(assignment):
    """    Chooses which variable to try values for next and returns said variable, or null if no unassigned variables found    """
    for cell in assignment:
        if cell.value == 0:
            return cell
    return None

def is_consistent(constraints, assignment, cell, value):
    """    Returns true if the assignment doesn't contain any duplicates, otherwise false    """
    # Add the new value to the assignment before checking
    temp_assignment = assignment.copy()
    temp_assignment[cell.position].value = value
    for constraint in constraints:
        if not constraint.check_satisfied(temp_assignment):
            #print("Assignment is not consistent!")
            return False
    #print("Assignment is consistent")
    return True

def propagate(cell, value, assignment):
    """
        Updates the domain of all variables based on the supplied assignment. 
        Returns null if this variable-assignment combo results in an invalid sudoku
    """
    if cell.update_value(value) == False:
        return None

    rowindex = math.floor((cell.position - (cell.position % 9)) / 9)
    colindex = (cell.position % 9)
    for u in units:
        if u.__contains__(cell.position):
            unitindex = units.index(u)

    for c in assignment:
        if rows[rowindex].__contains__(c.position) and not c.position == cell.position:
            if not c.remove_from_domain(value):
                return None
        if columns[colindex].__contains__(c.position) and not c.position == cell.position:
            if not c.remove_from_domain(value):
                return None
        if units[unitindex].__contains__(c.position) and not c.position == cell.position:
            if not c.remove_from_domain(value):
                return None

    return assignment

def restore(cell, assignment, value=0):
    """
        Repopulates the domains of the appropriate cells in the assignment.
        Restores previous value of cell.
    """
    cell.restore_previous_value()

    rowindex = math.floor((cell.position - (cell.position % 9)) / 9) 
    colindex = (cell.position % 9)
    for u in units:
        if u.__contains__(cell.position):
            unitindex = units.index(u)

    for c in assignment:
        if rows[rowindex].__contains__(c.position) and c.fixed==False:
            c.add_to_domain(value)
        if columns[colindex].__contains__(c.position) and c.fixed==False:
            c.add_to_domain(value)
        if units[unitindex].__contains__(c.position) and c.fixed==False:
            c.add_to_domain(value)

    return assignment

def backtrack(constraints, assignment):
    """    Main backtracking algorithm    """
    #if assignment is complete then return assignment
    if is_complete(constraints, assignment):
        return assignment
    #var = select_unassigned_variable
    var = select_unassigned_variable(assignment)
    #for each value in order-domain-values(var,assignment,csp):
    if not var is None:
        for value in list(var.domain):
            #if value is consistent with assignment then:
            #print("Trying", value, "in", var.position)
            if is_consistent(constraints, assignment, var, value):
                #add var=value to assignment
                new_assignment = copy.deepcopy(assignment)
                new_assignment[var.position].value = value
                #inferences <- inference(csp,var,value)
                new_assignment = propagate(var, value, assignment)
                #if inferences != failure then:
                if new_assignment is not None:
                    #add inferences to assignment
                    #result <- backtrack(assignment, csp)
                    result = backtrack(constraints, copy.deepcopy(new_assignment))
                    #if result != failure then:
                    if not result == []:
                        #return result
                        return result
            #remove var=value and inferences from assignment
            #---the old assignment is never used if new_assignment fails---#
    #return failure
    return []


```
