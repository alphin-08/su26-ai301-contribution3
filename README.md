**Contribution Number:** 3 

**Student:** Alphin Shajan 

**Issue:** https://github.com/carlos-emr/carlos/issues/2757

**Status:** Phase IV

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

1. Log in to the application as a provider account that has the prevention write privilege so you have the correct permissions to reproduce the issue.
2. Open any patient encounter and navigate to the Preventions panel where you will find the standard form used to add a prevention.
3. Use your browser developer tools to edit the form before you submit it so you can manipulate the hidden fields.
4. Submit the POST request but completely omit the demographic number parameter or change the value to letters instead of numbers.
5. Observe that the request fails with an internal server error instead of returning a validation message to the user.
6. Check the server log to find a number format exception originating from the action file when the system tries to parse the missing or bad input.


   
### Reproduction Evidence

- **Commit showing reproduction:** 
- **Screenshots/logs:** Not applicable since this is a code convention issue and not a visible crash.
- **My findings:** This crash is a server error and not just a style issue. The demographic number is read without any validation and passed straight into the validate method where the system tries to parse it. The Struts framework shows this unhandled exception as a server error page. Other lines in the file share this exact same problem but they only run after the initial validation passes.
---

## Solution Approach

### Analysis

The root cause of the issue is that the demographic number is treated as a real input. It is read from the request and passed into the validation method without ever being checked for a missing value or incorrect format. Any missing or bad value throws an exception before any proper validation error can be recorded. The good news is that the validate method is already the perfect place to fix this problem. It returns a list of error messages and checks that list before moving forward. The user will get a clean validation message and the request will end normally if we reject a bad demographic number by adding an error to that list. This fix also protects the later code branches because they will only run if the validation method guarantees the number is valid. The codebase already has a proven pattern for this exact situation in another file called the retail preventions action. We can copy that pattern to check for missing values and ensure the input only contains digits.


### Proposed Solution
I will match the existing solution from the other file to validate that the identifier is present and only contains digits. I will insert this new check at the very top of the validate method before the system tries to parse the integer. I will add a validation error to the returned list so the system forwards the user to a proper error screen instead of crashing. This keeps the fix small and consistent with the accepted pattern in the codebase.



### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** 
A missing or bad demographic number causes the system to throw an unhandled exception that surfaces to the user as a crash. The endpoint should reject this bad input gracefully rather than crashing the system.


**Match:**
I will match the existing solution from the other file to validate that the identifier is present and only contains digits. I will reuse that proven pattern rather than inventing a new one.


**Plan:** 
I will insert a check for missing values and numbers at the top of the validate method. I will add a validation error to the returned list so the system forwards the user to a proper error screen instead of crashing. I will also confirm this early return makes the later lines safe since they only run after a successful validation.


**Implement:** 
I will edit the specific Java file to add the guard while keeping the change limited to that single method. I will preserve the existing copyright header and follow the security conventions of the project.

**Review:**
I will verify that the new guard runs before any parsing happens. I will confirm that no sensitive patient information is placed in the public error message.

**Evaluate:**
I will add unit tests covering missing inputs and bad formatting to ensure the system returns a validation error instead of throwing an exception. I will also manually rerun the reproduction request to confirm it works properly.


---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: A missing value where demographic_no is null. The helper should return null and the code should add a validation error instead of crashing.
- [ ] Test case 2: A value that is not a number, like abc. The helper should return null so the request is rejected cleanly.
- [ ] Test case 3: An overflow value like 99999999999 that is all digits but too large to fit in an int. This is the case a simple digit check misses, and the helper should still return null instead of throwing an error.
- [ ] Test case 4: A normal valid id like 123. The helper should return the parsed number so the patient lookup can run.
- [ ] Test case 5: An empty value where demographic_no is an empty string. The helper should return null so an empty field is treated the same as a missing one.
- [ ] Test case 6: A value that mixes digits and letters, like 12a. The helper should return null because the value is not a clean number.
- [ ] Test case 7: The value 2147483648, which is exactly one past the largest number an int can hold. The helper should return null instead of throwing an error, which is another version of the overflow case.
- [ ] Test case 8: The value 2147483647, which is the exact largest number an int can hold. The helper should return that number, since this proves a valid value right at the edge still works correctly.

### Integration Tests

I did not add any integration tests for this change. The fix is a small piece of validation logic inside one method, and the unit tests already cover every input case directly, so a database backed integration test would not add real value here.

### Manual Testing

My main verification was running the unit test through the real Maven build inside the dev container, and it passed all eight cases. I did not do a full browser walkthrough of the reproduction steps, since the unit tests exercise the exact logic that was broken. If you want to add a browser based check later, the reproduction steps in the earlier section describe how to trigger the old crash.

---

## Implementation Notes

### Week [1] Progress
This week I focused on picking the issue and understanding it before writing any code. I read through the issue description and then read the feedback that a maintainer had left for the person who tried this issue before me. That feedback was the most useful part, because it explained exactly why the earlier attempt did not pass. The maintainer pointed out that a simple digit check is not enough, since a value that is all digits can still be too large to fit in an int and crash the same way. After I understood that, I opened the latest upstream version of the file and confirmed that the bug was still there. The unguarded parse call was still in the validate method, and there was no check in front of it, so I knew the issue was real and still worth fixing.


### Week [2] Progress
This week was mostly about getting assigned and getting my environment ready. The issue had first been claimed by someone else, but they closed their own pull request, which left the issue open again for anyone to take. Because of that I had to wait for a maintainer to assign it to me before I started, since I did not want to duplicate work that someone else might still be doing. Once I was assigned, I pulled the most recent changes from upstream so I was working on top of the latest code and not an old version. After my branch was up to date, I spent time reading the validate method and the execute method around it, and I looked closely at the RtlPreventions action that the issue pointed to as the example to follow. That gave me a clear plan for how I wanted to build the fix.


### Week [3] Progress
This week I actually wrote the fix and the test and got everything verified. I added a small helper method that checks the value and then parses it inside a try and catch, which means a missing value, a value that is not a number, and a value that is too large for an int all return a clean result instead of crashing. This directly handled the overflow case that the earlier attempt missed, which was the main thing the maintainer asked for. I also wrote a unit test with eight cases that cover all of those situations, including the overflow and the exact boundary values. I ran the whole thing through a real Maven build inside the dev container, and all eight cases passed with a clean build. After that I opened a pull request, which is now open for maintainer feedback and ready to be merged.




### Code Changes

- **Files modified:** src/main/java/io/github/carlos_emr/carlos/prevention/pageUtil/AddPrevention2Action.java was modified, and a new test file named AddPrevention2ActionParseDemographicNoUnitTest.java was added in the matching test folder.
- **Key commits:** https://github.com/alphin-08/carlos/commit/206fd74c6745f629779f9a389b546103987d37ef 
- **Approach decisions:** I added a small helper method called parseDemographicNo instead of writing the check inline, which made it much easier to test on its own. I followed the same check then parse pattern that the RtlPreventions action already uses, and I parsed the value inside a try and catch so the overflow case is handled. I chose not to touch the two similar lines in the execute method because validate now stops the request before those lines ever run, so they are already safe.
---

## Pull Request

**PR Link:** https://github.com/carlos-emr/carlos/pull/3300 

**PR Description:**

The validate() method in AddPrevention2Action called Integer.parseInt(demographic_no) without checking the value first. When the demographic_no request parameter is missing, request.getParameter(...) returns null, and when it is not a number, parseInt throws a NumberFormatException that nothing catches. Struts then turns that into a 500 Internal Server Error instead of showing the user a normal validation message. The same unguarded parsing also happens at lines 264 and 275 in the DHIR branch of execute().

This PR adds a small helper called parseDemographicNo(String). The helper first makes sure the value is there and contains only digits, and then it parses the value inside a try and catch. It returns null when the value is missing, is not a number, or is too big to fit in an int. The validate() method takes that null and adds an "Invalid or missing demographic_no" message to
the error list instead of crashing. Since execute() sends the user back to the form whenever validate() finds any error, the parsing at lines 264 and 275 is also safe now, so those lines did not need to change.

**Maintainer Feedback:** N/A

**Status:** Waiting for review

---

## Learnings & Reflections

### Technical Skills Gained


### Challenges Overcome


### What I'd Do Differently Next Time


 
---

## Resources Used

- Link to helpful documentation: https://github.com/carlos-emr/carlos/blob/develop/CONTRIBUTING.md, https://github.com/carlos-emr/carlos/blob/develop/CLAUDE.md, https://github.com/carlos-emr/carlos/blob/develop/.devcontainer/README.md
