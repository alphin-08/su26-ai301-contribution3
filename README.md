**Contribution Number:** 3 

**Student:** Alphin Shajan 

**Issue:** https://github.com/carlos-emr/carlos/issues/2757

**Status:** Phase I

---

## Why I Chose This Issue
I picked this issue because it is a well defined bug with a clear suggested fix already included in the description. The scope is limited to one method in one file, which makes it easy to understand and test without needing to dig through a large portion of the codebase. I also wanted to get some experience with input validation bugs since they come up a lot in real software development, and this one is a good example of how a missing null check can turn into a server error that the user should never have to see.


---

## Understanding the Issue

### Problem Description

AddPrevention2Action.validate() calls Integer.parseInt(demographic_no) at line 314 without first checking whether demographic_no is null or contains a non-numeric value. When a request comes in without that parameter, request.getParameter("demographic_no") returns null, and passing null directly into Integer.parseInt() causes a NumberFormatException to be thrown. Because the validate method has no exception handler around this call, the exception returns a 500 Internal Server Error to the user instead of a proper validation message. The same unguarded parseInt call also appears at lines 264 and 275, meaning those lines have the same risk.


### Expected Behavior

When a request comes in with a missing or non-numeric parameter, the validate method should catch that early and add a clear validation error to the result list. The user should see a proper validation message rather than a 500 error, and the application should continue handling the request normally without throwing an unhandled exception.


### Current Behavior

If the demographic_no parameter is missing from the request, Integer.parseInt(null) throws a NumberFormatException inside validate(). Since there is no exception handler around that call, the error returns a 500 Internal Server Error. The same thing would happen if the parameter is present but contains a non-numeric string like letters or special characters.


### Affected Components

The main file being changed is src/main/java/io/github/carlos_emr/carlos/prevention/pageUtil/AddPrevention2Action.java, specifically the validate() method at lines 264, 275, and 314. The fix will follow the same validation pattern already used in RtlPreventions2Action at lines 109 to 120, which uses a regex check and a try catch before making any parseInt call.

---

## Reproduction Process

### Environment Setup

I did not have to do any further environment setup other than what I did for my last issue since it is the same project. I set up the development environment using Docker Desktop and VS Code with the Dev Containers extension on Windows. The biggest challenge was enabling WSL integration in Docker Desktop, since without it the container could not find the Docker daemon. Once that was enabled under Settings, Resources, and WSL Integration, everything connected properly. I also had to run git config --global --add safe.directory /workspace inside the container because Git flagged an ownership mismatch that was blocking the build.

### Steps to Reproduce




   
### Reproduction Evidence

- **Commit showing reproduction:** 
- **Screenshots/logs:** Not applicable since this is a code convention issue and not a visible crash.
- **My findings:**  
---

## Solution Approach

### Analysis




### Proposed Solution




### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** 


**Match:**


**Plan:** 


**Implement:** 

**Review:**

**Evaluate:**


---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: 
- [ ] Test case 2: 
- [ ] Test case 3:
- [ ] Test case 4:

### Integration Tests



### Manual Testing


---

## Implementation Notes

### Week [1] Progress



### Week [2] Progress


### Week [3[ Progress




### Code Changes

- **Files modified:** 
- **Key commits:**
- **Approach decisions:** 
---

## Pull Request

**PR Link:**

**PR Description:** 


**Maintainer Feedback:** 

**Status:**

---

## Learnings & Reflections

### Technical Skills Gained


### Challenges Overcome


### What I'd Do Differently Next Time


 
---

## Resources Used

- Link to helpful documentation: https://github.com/carlos-emr/carlos/blob/develop/CONTRIBUTING.md, https://github.com/carlos-emr/carlos/blob/develop/CLAUDE.md, https://github.com/carlos-emr/carlos/blob/develop/.devcontainer/README.md
