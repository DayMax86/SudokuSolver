University assignment for AI module - implement a sudoku solver using our choice of algorithm, in my case a depth-first search with backtracking and heuristics. The task only required the code to work for 9x9 grids but I allowed for any size.

The main code can be found below and has been taken from the second cell of the .ipynb file.

```
import math
import collections
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
    #Create fillRemainingGrid function that takes an index value and returns a boolean
    #--If the index is larger than the total number of cells (out of range) 
    #----return true
    #--If grid[index] is an empty cell
    #----For each value (n) in the domain 
    #------grid[index] = n
    #------If this new entry is valid with the rest of the grid
    #--------If fillRemainingGrid(next)
    #----------Return true
    #----grid[index] = 0
    #------Return false
    #--Else return fillRemainingGrid(next)
    
    if fillRemainingGrid(sudoku, 0):
        solved_sudoku = sudoku
    else:
        #print("Invalid sudoku!")
        solved_sudoku = np.array([[ -1 for x in range(0, getWidth(sudoku))] for y in range(0, getWidth(sudoku))])

    return solved_sudoku


def fillRemainingGrid(sudoku, index):

    if (index > (np.square(getWidth(sudoku)) - 1)): #Index out of range - no remaining cells
        return True
    
    domain = len(sudoku[0])

    r, c = indexToCoords(index)

    if (isEmptyCell(sudoku, r, c)):
        validEntries = getValidEntries(sudoku, r, c)
        for n in validEntries:
            sudoku[r,c] = n
            #print("Trying ", n, " in position (", r,",",c,")")
            if (isValid(sudoku, r, c)):
                if (fillRemainingGrid(sudoku, index + 1)):
                    return True
                
        sudoku[r, c] = 0
        #print("BACKTRACKING - replacing (",r, ",", c,") with 0" )
        #return False
    else:
        return (fillRemainingGrid(sudoku, index + 1))

def getValidEntries(sudoku, r, c):
    # Produce a list of possible values based on bounding row, column and square values.
    # This is more efficient than trying every possible value in the domain for each empty cell
    validEntries = set(range(1, getWidth(sudoku)+1, 1))
    square = getSquare(sudoku, r, c)
    squareValues = []
    for squareRow in square:
        for value in squareRow:
            squareValues.append(value)
    row = getRow(sudoku, r)
    column = getColumn(sudoku, c)
    validEntries = validEntries.difference(squareValues, row, column)
    #print("ValidEntries list: ", validEntries)
    return validEntries

def indexToCoords(index):
    width = getWidth(sudoku)
    q, r = divmod(index, width)
    rowIndex = q
    columnIndex = r
    if rowIndex >= width:
        rowIndex = width-1
    return rowIndex, columnIndex

def isEmptyCell(sudoku, rowIndex, columnIndex):
    if (sudoku[rowIndex, columnIndex] == 0):
        return True
    else:
        return False
    

def isValid(sudoku, rowIndex, columnIndex):

    # Square
    square = getSquare(sudoku, rowIndex, columnIndex)
    duplicateSquare = 0
    for row in square:
        for cell in row:
            if sudoku[rowIndex, columnIndex] == cell:
                duplicateSquare += 1
                if duplicateSquare > 1:
                    #for r in square:
                    #    print(r)
                    #print("Invalid square! : ", cell, " present more than once")
                    return False
            
    # Row
    row = getRow(sudoku, rowIndex)
    tempRow = []
    for cell in row:
        if (cell>0):
            if (not (cell in tempRow)):
                tempRow.append(cell)
            else:
                #print("Invalid row! : ", row, " contains ", cell, " more than once")
                tempRow.clear()
                return False
    
    # Column
    column = getColumn(sudoku, columnIndex)
    tempColumn = []
    for cell in column:
        if (cell>0):
            if (not (cell in tempColumn)):
                tempColumn.append(cell)
            else:
                #print("Invalid column! : ", column, " contains ", cell, " more than once")
                tempColumn.clear()
                return False
    
    #print("Valid entry!")
    return True

def getWidth(sudoku):
    return len(sudoku[0])

def getRow(sudoku, index):
    r = sudoku[index]
    return r
    
def getColumn(sudoku, columnIndex):
    c = collections.deque([])
    for row in sudoku:
        c.append(row[columnIndex])
    return c

def getSquare(sudoku, rowIndex, columnIndex):
    w = int(math.floor(math.sqrt(getWidth(sudoku)))) # Width of an inner square (should always be whole number but floored just in case)
    # RowIndex, columnIndex MOD w -> the quotient * w is the index of the corner of the inner square
    squareStartingRow, r1 = divmod(rowIndex, w)
    squareStartingColumn, r2 = divmod(columnIndex, w)
    squareStartingRow = squareStartingRow * w
    squareStartingColumn = squareStartingColumn * w
    # Construct a smaller 2D array 
    square = collections.deque([])
    # W times, take sudoku[rowQuotient + i], i starts at 0
    for i in range(0, (w)):
        square.append(sudoku[squareStartingRow + i, squareStartingColumn: squareStartingColumn + w])
    #print("mini square in location (",rowIndex,",",columnIndex,")\n",square)
    return square

```
