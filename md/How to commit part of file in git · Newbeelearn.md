> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [newbeelearn.com](https://newbeelearn.com/blog/git-commit-part-of-file/)

> Explains how to commit part of file in git with Emacs magit and git cli

In [Magit](https://magit.vc/), the Git interface for Emacs, you can commit part of a file using the following commands:

**Stage Specific Hunks:**

*   Navigate to the hunks you want to stage using the arrow keys or mouse.
    
    ![](https://newbeelearn.com/img/git-commit-part-of-file-hunk.png)
*   Press s to stage the changes in the current hunk (shown as "Hunk 1" in image above).
*   Repeat it for each hunk you want to stage.
    
    ![](https://newbeelearn.com/img/git-commit-part-of-file-hunk-staged.png)

Explanation: This command stages specific hunks (chunks of changes) within the file, allowing you to selectively include changes in the next commit.

You are not limited to a single file either, if you have multiple changes in multiple files you can select combination of hunks using the same procedure.

**Stage Specific Lines:**

*   Move the cursor to highlight the desired lines.
    
    ![](https://newbeelearn.com/img/git-commit-part-of-file-line.png)
*   Select the line(s) you want to stage by placing the cursor on the first line and pressing Ctrl-SPC (Space) to start the selection.
*   Press s to stage the changes on the selected line(s).
    
    ![](https://newbeelearn.com/img/git-commit-part-of-file-line-staged.png)

Explanation: You can choose to stage specific lines of code by first selecting them. Start the selection by placing the cursor on the initial line and pressing Ctrl-SPC. Then, move the cursor to encompass the lines you wish to stage. After selecting the desired lines, use the staging command (s) to include their changes in the commit.

You are not limited to a single line, if you have multiple lines in multiple files you can select combination of lines using the same procedure. If the change encompasses multiple lines you can select region as well and use same staging procedure to stage regions.