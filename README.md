# claude-coding-companion
Coding Companion: Claude for Large Codebases

# AI Code Assistant: Revolutionizing Large-Scale Codebase Management

Claude AI continues to lead in AI-powered code generation, but faces significant challenges when working with large codebases due to token window limitations. This becomes particularly critical for single-file applications where traditional copy-paste approaches for code analysis become impractical as files grow beyond manageable sizes. The conventional method of feeding entire files to AI for feature development or debugging breaks down when codebases exceed token limits.

## The Solution: Intelligent Code Segmentation

This AI Code Assistant tool fundamentally transforms how AI interacts with large codebases by implementing a sophisticated JSON-based code extraction system. Rather than processing entire files, the tool enables AI to selectively request and analyze specific code segments through a comprehensive indexing mechanism.

## Key Technical Innovations

### Advanced Code Parsing Engine
The tool employs sophisticated regex patterns and AST-like analysis to accurately identify and extract:
- PHP functions and classes
- JavaScript functions
- HTML elements and CSS selectors
- Comment blocks
- Switch cases and handler functions
- Database table definitions

It maintains context awareness by including configurable lines before and after each extracted segment.

### Comprehensive Codebase Intelligence
The system generates detailed indexes that include:
- **Function call graphs** - Complete mapping of function dependencies
- **Unused code analysis** - Identifies functions at multiple dependency levels
- **Duplicate function detection** - Prevents code conflicts and redundancy
- **Handler mapping** - Complete inventory of POST/GET handlers
- **Structural analysis** - HTML components and database schemas

### API-Driven Architecture
- **Web Interface** - User-friendly GUI for interactive code exploration
- **REST API** - Programmatic access through JSON requests
- **Seamless Integration** - Compatible with AI workflows and automated analysis pipelines

### Smart Context Management
Unlike simple text extraction, the tool understands code structure:
- Accurate function boundary detection using brace matching
- Nested structure handling
- Intelligent context provision for AI decision-making
- Relationship and dependency awareness

## Performance Validation

Experimental comparisons with Cursor and other automated code writing tools demonstrate **superior efficiency** in developing new PHP, HTML, CSS, and JavaScript projects. The tool's ability to provide precise, contextual code segments enables AI to make more informed decisions while requiring significantly fewer tokens than traditional full-file approaches.

## Impact on Development Workflows

This approach addresses the fundamental scalability issue in AI-assisted development, enabling continued productivity growth as codebases expand. By providing AI with precisely the information it needs—no more, no less—the tool optimizes both performance and accuracy while maintaining the contextual awareness necessary for intelligent code modification and feature development.

## Key Features

- ✅ **Token Efficiency** - Dramatically reduces token usage for large files
- ✅ **Precise Extraction** - Surgical code analysis instead of brute-force processing
- ✅ **Context Preservation** - Maintains code relationships and dependencies
- ✅ **Duplicate Detection** - Prevents and resolves code conflicts
- ✅ **Usage Analysis** - Identifies unused and potentially problematic code
- ✅ **Multi-Language Support** - PHP, JavaScript, HTML, CSS, and SQL
- ✅ **API Integration** - Programmatic access for AI workflows

## Usage Example

```json
{
 "php": ["process_data", "validate_user_input"],
 "php_class": ["DatabaseManager"],
 "js": ["updateUI"],
 "html": ["#main-content", ".btn-submit", "forms"],
 "comments": ["User Profile Section"],
 "blocks": ["ajax_handler"],
 "post_handlers": ["save_user_profile"],
 "get_handlers": ["fetch_user_data"],
 "switch_case": ["delete_record"],
 "create_tables": ["users"]
}

```php
<?php
/**
 * =================================================================
 * VYROX AI CODE ASSISTANT - COMPLETE & FINAL VERSION (v5.9.0 - MODIFIED & REVIEWED)
 * =================================================================
 * This single-file tool provides a comprehensive UI and API for extracting
 * and analyzing code from a target file (index.php). This version has a
 * re-engineered backend to ensure stability and correct handling of all
 * extraction types, including multiple occurrences of the same function.
 *
 * FEATURES:
 * - Refactored with helper functions to eliminate code duplication (DRY).
 * - Extracts ALL occurrences of PHP/JS functions, PHP classes, HTML selectors, generic blocks, and handlers.
 * - Extracts logical HTML sections demarcated by comments (e.g., <!-- Section -->).
 * - Accurately detects and reports duplicate function names (regardless of parameters).
 * - Provides clear, actionable instructions within the output text to fix duplicates.
 * - Live line counts for both the codebase index and the extraction results.
 * - Clear UI for configuring context lines (the number of lines before/after a result).
 * - Modern UI with Flexbox and enhanced JavaScript for a professional experience.
 * - Robust "auto-extract on paste" functionality.
 * - Comprehensive codebase index for easy discoverability of extractable items.
 * - Professional-grade code with PHPDoc comments and clear organization.
 *
 * v5.8.0 ENHANCEMENTS:
 * - Added a new API feature to request the full codebase index.
 * - A GET request to `?api=1&index=1` now returns the complete index as a JSON object.
 * - Refactored index generation into a dedicated `generateCodebaseIndex()` function for reusability.
 * - Updated API documentation with instructions for the new index endpoint.
 *
 * v5.9.0 ENHANCEMENTS (USER REQUEST):
 * - Added a new API command to request comprehensive API instructions and examples.
 * - A GET request to `?api=1&instructions=1` now returns the full API documentation.
 * - Created a `getApiInstructions()` function to centralize the documentation text.
 * - The on-file documentation is now generated by this function to ensure it's always in sync with the API response.
 */

// =================================================================
// HELPER FUNCTIONS
// =================================================================

/**
 * Formats the output for a found code block with context lines.
 * @param array $lines All lines of the source file.
 * @param int|null $start The starting line index of the found block.
 * @param int|null $end The ending line index of the found block.
 * @param int $contextLines Number of context lines before and after.
 * @param string $title A title for this extracted block.
 * @return string Formatted code block string.
 */
function formatContextOutput($lines, $start, $end, $contextLines, $title) {
    if ($start === null || $end === null) return '';
    $contextStart = max(0, $start - $contextLines);
    $contextEnd = min(count($lines) - 1, $end + $contextLines);
    $result = "$title\n";
    $result .= "// Context: {$contextLines} lines before/after.\n";
    $result .= str_repeat("-", 40) . "\n";
    for ($k = $contextStart; $k <= $contextEnd; $k++) {
        $prefix = ($k >= $start && $k <= $end) ? ">>> " : "    ";
        $result .= $prefix . (isset($lines[$k]) ? rtrim($lines[$k]) : '') . "\n";
    }
    return $result;
}

/**
 * Finds the line number of the closing brace for a block starting at $startLine.
 * Ignores braces within simple strings and single-line comments.
 * @param array $lines Code lines.
 * @param int $startLine Line index where the block's opening brace is expected or before.
 * @return int|null Ending line index or null if not found.
 */
function findBlockEndByBraces($lines, $startLine) {
    $braceCount = 0; $inBlock = false;
    for ($j = $startLine; $j < count($lines); $j++) {
        // Simple removal of common comment and string types for brace counting.
        // Not foolproof for complex nested or escaped structures or multi-line comments.
        $line = preg_replace('/(\/\/.*|#.*|\'[^\']*\'|"[^"]*")/', '', $lines[$j]);
        if (strpos($line, '{') !== false) {
            $braceCount += substr_count($line, '{');
            $inBlock = true;
        }
        if (strpos($line, '}') !== false) {
            $braceCount -= substr_count($line, '}');
        }
        if ($inBlock && $braceCount === 0) return $j;
    }
    return null;
}

/**
 * Extracts function names and their body content.
 * @param string $content The code content to parse.
 * @param string $lang 'php' or 'js'.
 * @return array A map of [functionName => bodyContentString].
 */
function get_function_bodies_with_names($content, $lang = 'php') {
    $lines = explode("\n", $content);
    $functions_data = [];
    $pattern_func_def = $lang === 'php' ? '/^\s*function\s+(&?\s*([a-zA-Z_\x7f-\xff][a-zA-Z0-9_\x7f-\xff]*))\s*\(/im' : '/^\s*(?:async\s+)?function\s+([a-zA-Z_$][a-zA-Z0-9_$]*)\s*\(/im';

    for ($i = 0; $i < count($lines); $i++) {
        if (preg_match($pattern_func_def, $lines[$i], $match)) {
            $functionName = $lang === 'php' ? trim(str_replace('&', '', $match[2])) : $match[1];
            
            $func_def_line_idx = $i;
            $body_open_line_idx = -1;

            // Find the line with the opening brace for the function body
            for ($j = $func_def_line_idx; $j < count($lines); $j++) {
                $current_line_content = $lines[$j];
                // Clean line for brace detection (ignores simple comments/strings)
                $cleaned_line = preg_replace('/(\/\/.*|#.*|\/\*.*?\*\/|\'[^\']*\'|"[^"]*")/', '', $current_line_content);
                
                $brace_pos = strpos($cleaned_line, '{');
                if ($brace_pos !== false) {
                    if ($j == $func_def_line_idx) { // Brace on the same line as "function foo()"
                        $sig_end_pos = strrpos($cleaned_line, ')'); // Find the ')' of the signature
                        if ($sig_end_pos !== false && $brace_pos > $sig_end_pos) { // Ensure brace is after ')'
                            $body_open_line_idx = $j;
                            break;
                        }
                    } else { // Brace is on a subsequent line
                        $body_open_line_idx = $j;
                        break;
                    }
                }
                // If another function definition starts before finding a '{' for the current one
                if ($j > $func_def_line_idx && preg_match($pattern_func_def, $current_line_content, $next_match)) {
                    $nextFuncName = $lang === 'php' ? trim(str_replace('&', '', $next_match[2])) : $next_match[1];
                    if ($nextFuncName !== $functionName) break; // Stop if it's a different function
                }
            }

            if ($body_open_line_idx === -1) continue; // No opening brace found

            $endLine = findBlockEndByBraces($lines, $body_open_line_idx);
            if ($endLine === null) continue; // No closing brace found

            // Extract lines between the opening and closing braces
            $body_lines_arr = array_slice($lines, $body_open_line_idx + 1, $endLine - $body_open_line_idx - 1);
            if (!isset($functions_data[$functionName])) { // Store first occurrence
                 $functions_data[$functionName] = implode("\n", $body_lines_arr);
            }
            $i = $endLine; // Continue search after this function
        }
    }
    return $functions_data;
}

/**
 * Extracts all unique case values from switch statements in the content.
 * @param string $content The code content to parse.
 * @return array A sorted array of unique case values.
 */
function extractAllCaseValues($content) {
    $caseValues = [];
    // This regex captures the string literal inside a case statement.
    $pattern = '/case\s+[\'"]([^\'"]+)[\'"]\s*:/i';
    preg_match_all($pattern, $content, $matches);
    if (!empty($matches[1])) {
        $caseValues = array_unique($matches[1]);
        sort($caseValues);
    }
    return $caseValues;
}


// =================================================================
// EXTRACTION & ANALYSIS FUNCTIONS (ALL NOW RETURN ARRAYS)
// =================================================================

function countTotalLines($content) { return $content ? substr_count($content, "\n") + 1 : 0; }

function extractPhpFunction($content, $functionName, $contextLines) {
    $lines = explode("\n", $content); $foundBlocks = [];
    for ($i = 0; $i < count($lines); $i++) {
        if (preg_match('/function\s+' . preg_quote($functionName, '/') . '\s*\(/i', $lines[$i])) {
            $functionEnd = findBlockEndByBraces($lines, $i) ?? $i; // Default to current line if no block found
            $foundBlocks[] = formatContextOutput($lines, $i, $functionEnd, $contextLines, "// PHP FUNCTION: $functionName");
            $i = $functionEnd; 
        }
    } return $foundBlocks;
}

function extractPhpClass($content, $className, $contextLines) {
    $lines = explode("\n", $content); $foundBlocks = [];
    $pattern = '/^\s*(?:abstract\s+|final\s+)*class\s+' . preg_quote($className, '/') . '(\s|\{)/i';
    for ($i = 0; $i < count($lines); $i++) {
        if (preg_match($pattern, $lines[$i])) {
            $classEnd = findBlockEndByBraces($lines, $i) ?? $i;
            $foundBlocks[] = formatContextOutput($lines, $i, $classEnd, $contextLines, "// PHP CLASS: $className");
            $i = $classEnd;
        }
    }
    return $foundBlocks;
}

function extractJsFunction($content, $functionName, $contextLines) {
    $lines = explode("\n", $content); $foundBlocks = []; $inScript = false;
    for ($i = 0; $i < count($lines); $i++) {
        if (preg_match('/<script/i', $lines[$i]) && stripos($lines[$i], 'src=') === false) $inScript = true;
        if ($inScript && preg_match('/(?:async\s+)?function\s+' . preg_quote($functionName, '/') . '\s*\(/i', $lines[$i])) {
            $funcDefLine = $i;
            $bodyOpenLine = -1;

            // Revised logic to find opening brace for JS function
            for($k = $funcDefLine; $k < count($lines); $k++){
                $current_js_line = $lines[$k];
                $cleaned_js_line = preg_replace('/(\/\/.*|\/\*.*?\*\/|\'[^\']*\'|"[^"]*")/', '', $current_js_line);
                $brace_pos = strpos($cleaned_js_line, '{');

                if($brace_pos !== false){
                    if ($k === $funcDefLine) { // Brace on same line as function def
                        $sig_end_pos = strrpos($cleaned_js_line, ')');
                        if ($sig_end_pos !== false && $brace_pos > $sig_end_pos) {
                           $bodyOpenLine = $k; break;
                        }
                    } else { // Brace on a subsequent line
                        $bodyOpenLine = $k; break;
                    }
                }
                if(preg_match('/<\/script>/i', $current_js_line) && $k > $funcDefLine) break; 
            }

            if ($bodyOpenLine !== -1) {
                $functionEnd = findBlockEndByBraces($lines, $bodyOpenLine) ?? $i;
                $foundBlocks[] = formatContextOutput($lines, $funcDefLine, $functionEnd, $contextLines, "// JAVASCRIPT FUNCTION: $functionName");
                $i = $functionEnd; 
            } else {
                 // Function definition found, but opening brace for body not clearly identified or not present
                 $foundBlocks[] = formatContextOutput($lines, $funcDefLine, $funcDefLine, $contextLines, "// JAVASCRIPT FUNCTION (declaration or no body found): $functionName");
            }
        }
        if (preg_match('/<\/script>/i', $lines[$i])) $inScript = false;
    } return $foundBlocks;
}

function extractCommentBlock($content, $commentText, $contextLines) {
    $allLines = explode("\n", $content); $startLine = null; $endLine = null;
    $startPattern = '/<!--\s*' . preg_quote($commentText, '/') . '\s*-->/i';
    for ($i = 0; $i < count($allLines); $i++) {
        if (preg_match($startPattern, $allLines[$i])) {
            $startLine = $i;
            // Try to find the end before the next comment or a significant closing tag
            for ($j = $i + 1; $j < count($allLines); $j++) {
                if (strpos($allLines[$j], '<!--') !== false) { $endLine = $j - 1; break; }
            }
            if ($endLine === null) { // If no next comment, try to find a structural end
                for ($k = count($allLines) - 1; $k > $i; $k--) {
                    if (strpos($allLines[$k], '</') !== false) { $endLine = $k; break; }
                }
            }
            if ($endLine === null) $endLine = count($allLines) - 1; // Default to end of file
            break; 
        }
    }
    return ($startLine !== null) ? [formatContextOutput($allLines, $startLine, $endLine, $contextLines, "// HTML COMMENT BLOCK: '$commentText'")] : [];
}

function extractHtmlContent($content, $htmlSelector, $contextLines) {
    $lines = explode("\n", $content); $foundBlocks = [];
    if (in_array(substr($htmlSelector, 0, 1), ['.', '#'])) { // Class or ID selector
        $type = (substr($htmlSelector, 0, 1) === '.') ? 'class' : 'id';
        $name = substr($htmlSelector, 1);
        $pattern = '/<[^>]*' . $type . '="[^"]*' . preg_quote($name, '/') . '[^"]*"[^>]*>/i';
        for ($i = 0; $i < count($lines); $i++) {
            if (preg_match($pattern, $lines[$i])) {
                $foundBlocks[] = formatContextOutput($lines, $i, $i, $contextLines, "// HTML " . strtoupper($type) . ": " . $htmlSelector);
            }
        }
        return $foundBlocks;
    }
    // Tag-based selectors
    switch ($htmlSelector) {
        case 'forms':
            for ($i = 0; $i < count($lines); $i++) {
                if (preg_match('/<form[^>]*>/i', $lines[$i])) {
                    $formEnd = $i;
                    for ($j = $i + 1; $j < count($lines); $j++) {
                        if (preg_match('/<\/form>/i', $lines[$j])) { $formEnd = $j; break; }
                    }
                    $foundBlocks[] = formatContextOutput($lines, $i, $formEnd, $contextLines, "// HTML FORMS");
                    $i = $formEnd; // Skip to end of this form
                }
            } return $foundBlocks;
        case 'buttons':
             for ($i = 0; $i < count($lines); $i++) {
                 if (preg_match('/<button[^>]*>/i', $lines[$i])) {
                     $foundBlocks[] = formatContextOutput($lines, $i, $i, $contextLines, "// HTML BUTTONS");
                 }
             }
             return $foundBlocks;
        case 'head':
            preg_match('/<head[^>]*>(.*?)<\/head>/is', $content, $headMatch);
            return $headMatch ? ["// HTML HEAD CONTENT\n" . str_repeat("-", 40) . "\n" . trim($headMatch[1])] : [];
    }
    return [];
}

function extractCodeBlock($content, $blockType, $contextLines) {
    $lines = explode("\n", $content);
    $foundBlocks = []; // Initialize to hold multiple blocks

    switch ($blockType) {
        case 'php_opening':
            for ($i = 0; $i < count($lines); $i++) {
                if (strpos($lines[$i], '<?php') !== false) {
                    $foundBlocks[] = formatContextOutput($lines, $i, $i, $contextLines, "// PHP OPENING TAG");
                }
            }
            return $foundBlocks;

        case 'ajax_handler': // A specific pattern for AJAX handling
            for ($i = 0; $i < count($lines); $i++) {
                if (preg_match('/if \(isset\(\$_(GET|POST|REQUEST)\[[\'"](ajax|action)[\'"]\]\)\)/', $lines[$i])) {
                    $blockStart = $i;
                    $blockEnd = findBlockEndByBraces($lines, $blockStart);
                    if ($blockEnd === null) { // Fallback if no clear brace structure
                        $tempEnd = $blockStart;
                        for ($j = $blockStart + 1; $j < min(count($lines), $blockStart + 15); $j++) {
                            if (trim($lines[$j]) === '}') { $tempEnd = $j; break; }
                            if (preg_match('/(exit|die)\s*;?$/i', trim($lines[$j]))) { $tempEnd = $j; break; }
                            $tempEnd = $j;
                        }
                        $blockEnd = $tempEnd;
                    }
                    $foundBlocks[] = formatContextOutput($lines, $blockStart, $blockEnd, $contextLines, "// AJAX HANDLER BLOCK (Pattern Detected)");
                    $i = $blockEnd; // Skip past this block
                }
            }
            return $foundBlocks;
    }
    return [];
}

function extractHandler($content, $handlerName, $type, $contextLines) {
    $lines = explode("\n", $content);

    // Search for switch/case handlers
    $searchPatternSwitch = '/case\s+[\'"]' . preg_quote($handlerName, '/') . '[\'"]\s*:/i';
    for ($i = 0; $i < count($lines); $i++) {
        if (preg_match($searchPatternSwitch, $lines[$i])) {
            $caseStart = $i;
            $isCorrectMethod = true; // Assume correct unless proven otherwise
            $switchVar = '';
            // Look upwards for the switch statement to verify method (POST/GET)
            for ($s = $i - 1; $s >= 0; $s--) {
                if (preg_match('/switch\s*\((.*?)\)/i', $lines[$s], $switchMatch)) {
                    $switchVar = trim($switchMatch[1]);
                    if (strtoupper($type) === 'POST' && stripos($switchVar, '$_POST') === false && stripos($switchVar, '$_REQUEST') === false) {
                        if (stripos($switchVar, '$_GET') !== false) $isCorrectMethod = false; // It's a GET switch
                    } elseif (strtoupper($type) === 'GET' && stripos($switchVar, '$_GET') === false && stripos($switchVar, '$_REQUEST') === false) {
                         if (stripos($switchVar, '$_POST') !== false) $isCorrectMethod = false; // It's a POST switch
                    }
                    break;
                }
                if (strpos($lines[$s], '}') !== false) break; // End of a block, likely enclosing switch
            }

            if (!$isCorrectMethod) continue; // Skip if method type doesn't match

            $caseEnd = $i;
            for ($j = $i + 1; $j < count($lines); $j++) {
                if (preg_match('/(break|case|default)\b/i', $lines[$j])) {
                    $caseEnd = preg_match('/break/i', $lines[$j]) ? $j : $j - 1; // Include break line or line before next case/default
                    break;
                }
                 if ($j == count($lines) - 1) $caseEnd = $j; // Reached end of file
            }
            return [formatContextOutput($lines, $caseStart, $caseEnd, $contextLines, "// " . strtoupper($type) . " HANDLER (CASE): $handlerName")];
        }
    }
    
    // Search for if-based handlers
    $method = strtoupper($type);
    // This regex looks for: if (isset($_METHOD['action']) && $_METHOD['action'] === 'handler') OR if ($_METHOD['action'] === 'handler')
    $searchPatternIf = '/if\s*\((?:isset\(\$_' . $method . '\[[\'"](?:action|operation|handler)[\'"]\])\s*&&\s*)?\$_' . $method . '\[[\'"](?:action|operation|handler)[\'"]\]\s*===?\s*[\'"]' . preg_quote($handlerName, '/') . '[\'"]\s*\)/i';
    for ($i = 0; $i < count($lines); $i++) {
        if (preg_match($searchPatternIf, $lines[$i])) {
            $blockStart = $i;
            $blockEnd = findBlockEndByBraces($lines, $i);
            if ($blockEnd === null) { // Fallback if no clear brace structure
                for ($j = $i + 1; $j < count($lines); $j++) {
                    $trimmedLine = trim($lines[$j]);
                    if (empty($trimmedLine) || strpos($trimmedLine, '//') === 0) continue; 
                    if (stripos($trimmedLine, 'exit') === 0 || stripos($trimmedLine, 'die') === 0 || strpos($trimmedLine, ';') !== false) { 
                        $blockEnd = $j;
                        break;
                    }
                    // If another control structure starts or a block closes, end before it
                    if (preg_match('/^(if|else|elseif|switch|foreach|for|while)\b/i', $trimmedLine) || strpos($trimmedLine, '}') === 0) { 
                        $blockEnd = $j - 1;
                        break;
                    }
                }
                if ($blockEnd === null || $blockEnd < $blockStart) $blockEnd = $blockStart + 1; // Default to a line or two if nothing else found
                $blockEnd = min($blockEnd, count($lines) -1);
            }
            return [formatContextOutput($lines, $blockStart, $blockEnd, $contextLines, "// " . strtoupper($type) . " HANDLER (IF): $handlerName")];
        }
    }
    return []; // No handler found
}

function extractTableDefinition($content, $tableName, $contextLines) {
    $lines = explode("\n", $content);
    // Looks for array key '$tableName' => 'CREATE TABLE ...'
    for ($i = 0; $i < count($lines); $i++) {
        if (preg_match('/[\'"]' . preg_quote($tableName, '/') . '[\'"]\s*=>\s*[\'"]CREATE TABLE/i', $lines[$i])) {
            $tableStart = $i; $tableEnd = $i;
            // Assumes SQL string ends with ')",' or ')"' which is specific to certain formatting
            for ($j = $i; $j < count($lines); $j++) {
                if (strpos($lines[$j], ')",') !== false || strpos($lines[$j], ')"') !== false) {
                    $tableEnd = $j; break;
                }
                if ($j == count($lines) - 1) $tableEnd = $j; // Reached end of file
            }
            return [formatContextOutput($lines, $tableStart, $tableEnd, $contextLines, "// CREATE TABLE DEFINITION: $tableName")];
        }
    } return [];
}

/**
 * Extracts a specific case block from a switch statement.
 * @param string $content The code content to parse.
 * @param string $caseValue The value of the case to find (e.g., 'create_admin').
 * @param int $contextLines Number of context lines before and after.
 * @return array An array of formatted code block strings for each match.
 */
function extractSwitchCase($content, $caseValue, $contextLines) {
    $lines = explode("\n", $content);
    $foundBlocks = [];
    $pattern = '/case\s+[\'"]' . preg_quote($caseValue, '/') . '[\'"]\s*:/i';

    for ($i = 0; $i < count($lines); $i++) {
        if (preg_match($pattern, $lines[$i])) {
            $caseStart = $i;
            $caseEnd = $i;
            // Find the end of this case block
            for ($j = $i + 1; $j < count($lines); $j++) {
                // End conditions: next case, default, break, or closing brace of the switch
                if (preg_match('/^\s*(case|default|break|return|exit|die)\b/i', $lines[$j]) || preg_match('/^\s*}/', $lines[$j])) {
                    // If it's a break, include the break line. Otherwise, end on the line before.
                    $caseEnd = preg_match('/(break|return|exit|die)/i', $lines[$j]) ? $j : $j - 1;
                    break;
                }
                if ($j === count($lines) - 1) { // Reached end of file
                    $caseEnd = $j;
                }
            }
            $foundBlocks[] = formatContextOutput($lines, $caseStart, $caseEnd, $contextLines, "// SWITCH CASE: '$caseValue'");
            $i = $caseEnd; // Continue search after this block
        }
    }
    return $foundBlocks;
}

function extractHandlers($content) { // Plural: extracts all handler names for indexing
    $handlers = ['post' => [], 'get' => []];
    $lines = explode("\n", $content);

    // Switch/case based handlers
    $currentSwitchVar = null;
    for ($i=0; $i < count($lines); $i++) {
        if (preg_match('/switch\s*\((.*?)\)/i', $lines[$i], $switchMatch)) {
            $currentSwitchVar = trim($switchMatch[1]);
        }
        if ($currentSwitchVar && preg_match('/case\s+[\'"]([^\'"]+)[\'"]\s*:/i', $lines[$i], $caseMatch)) {
            $handlerName = $caseMatch[1];
            // Check the switch variable for $_POST, $_GET, $_REQUEST
            if (stripos($currentSwitchVar, '$_POST') !== false || stripos($currentSwitchVar, '$_REQUEST') !== false || stripos($currentSwitchVar, '$_SERVER[\'REQUEST_METHOD\']') !== false) { 
                $handlers['post'][] = $handlerName;
            }
            if (stripos($currentSwitchVar, '$_GET') !== false || stripos($currentSwitchVar, '$_REQUEST') !== false) {
                 $handlers['get'][] = $handlerName;
            }
        }
    }
    
    // If-based handlers - Using safer, separate regex passes
    $p1 = '/if\s*\(\s*isset\(\$_(GET|POST)\[[\'"](?:action|operation|handler)[\'"]\]\)\s*&&\s*\$_(GET|POST)\[[\'"](?:action|operation|handler)[\'"]\]\s*===?\s*[\'"]([^\'"]+)[\'"]\s*\)/i';
    $m1 = [];
    if (@preg_match_all($p1, $content, $m1, PREG_SET_ORDER) !== false && !empty($m1)) {
        foreach ($m1 as $match) {
            $method = strtolower($match[2]);
            $handlerName = $match[3];
            $handlers[$method][] = $handlerName;
        }
    }

    $p2 = '/if\s*\(\$_(GET|POST)\[[\'"](?:action|operation|handler)[\'"]\]\s*===?\s*[\'"]([^\'"]+)[\'"]\s*\)/i';
    $m2 = [];
    if (@preg_match_all($p2, $content, $m2, PREG_SET_ORDER) !== false && !empty($m2)) {
        foreach ($m2 as $match) {
            $method = strtolower($match[1]);
            $handlerName = $match[2];
            $handlers[$method][] = $handlerName;
        }
    }

    $handlers['post'] = array_values(array_unique($handlers['post']));
    $handlers['get'] = array_values(array_unique($handlers['get']));
    sort($handlers['post']);
    sort($handlers['get']);
    return $handlers;
}

function detectDuplicateFunctions($extractedCode) {
    $phpCounts = []; $jsCounts = [];
    // Matches the title line generated by formatContextOutput
    preg_match_all('/^\/\/ PHP FUNCTION: (\w+)/m', $extractedCode, $phpMatches);
    foreach ($phpMatches[1] as $funcName) $phpCounts[$funcName] = ($phpCounts[$funcName] ?? 0) + 1;

    preg_match_all('/^\/\/ JAVASCRIPT FUNCTION: (\w+)/m', $extractedCode, $jsMatches);
    foreach ($jsMatches[1] as $funcName) $jsCounts[$funcName] = ($jsCounts[$funcName] ?? 0) + 1;
    
    return [
        'php' => array_keys(array_filter($phpCounts, fn($c) => $c > 1)),
        'js' => array_keys(array_filter($jsCounts, fn($c) => $c > 1))
    ];
}

function analyze_function_usage($file_content) {
    $analysis_results = [
        'php' => ['defined' => [], 'caller_map' => [], 'unused_level0' => [], 'called_by_unused_level0' => [], 'called_by_unused_level1' => []],
        'js' => ['defined' => [], 'caller_map' => [], 'unused_level0' => [], 'called_by_unused_level0' => [], 'called_by_unused_level1' => []]
    ];

    // --- PHP Analysis ---
    $php_function_bodies = get_function_bodies_with_names($file_content, 'php');
    $defined_php_functions = array_keys($php_function_bodies);
    sort($defined_php_functions);
    $analysis_results['php']['defined'] = $defined_php_functions;

    $php_caller_map = []; 
    $php_call_pattern = '/(?<!function\s)([a-zA-Z_\x7f-\xff][a-zA-Z0-9_\x7f-\xff]*)\s*\(/';

    foreach ($php_function_bodies as $caller_name => $body) {
        preg_match_all($php_call_pattern, $body, $calls_in_body);
        $unique_calls = array_unique($calls_in_body[1]);
        $valid_calls = array_intersect($unique_calls, $defined_php_functions);
        $php_caller_map[$caller_name] = array_values($valid_calls);
    }

    $handlers_data = extractHandlers($file_content);
    $php_entry_points = array_unique(array_merge($handlers_data['post'], $handlers_data['get']));

    preg_match_all($php_call_pattern, $file_content, $all_php_calls_match);
    $all_php_calls_in_file = array_unique($all_php_calls_match[1]);
    $all_php_calls_to_defined_functions = array_intersect($all_php_calls_in_file, $defined_php_functions);

    $calls_from_within_functions = [];
    foreach($php_caller_map as $callees) { 
        $calls_from_within_functions = array_merge($calls_from_within_functions, $callees);
    }
    $calls_from_within_functions = array_unique($calls_from_within_functions);

    $global_scope_calls = array_diff($all_php_calls_to_defined_functions, $calls_from_within_functions);
    $php_entry_points = array_unique(array_merge($php_entry_points, array_values($global_scope_calls)));
    $php_entry_points = array_values(array_intersect($php_entry_points, $defined_php_functions));

    $reachable_php = [];
    $queue_php = $php_entry_points;
    while(!empty($queue_php)) {
        $func = array_shift($queue_php);
        if (in_array($func, $defined_php_functions) && !in_array($func, $reachable_php)) {
            $reachable_php[] = $func;
            if (isset($php_caller_map[$func])) {
                foreach ($php_caller_map[$func] as $called_func) {
                    if (!in_array($called_func, $reachable_php)) { 
                        $queue_php[] = $called_func;
                    }
                }
            }
        }
    }
    sort($reachable_php);
    $analysis_results['php']['unused_level0'] = array_values(array_diff($defined_php_functions, $reachable_php));

    $called_by_u0_php = [];
    foreach ($analysis_results['php']['unused_level0'] as $u0_func) {
        if (isset($php_caller_map[$u0_func])) {
            $called_by_u0_php = array_merge($called_by_u0_php, $php_caller_map[$u0_func]);
        }
    }
    $analysis_results['php']['called_by_unused_level0'] = array_values(array_unique(array_intersect($called_by_u0_php, $defined_php_functions)));
    sort($analysis_results['php']['called_by_unused_level0']);

    $called_by_u1_php = [];
    foreach ($analysis_results['php']['called_by_unused_level0'] as $u1_caller_func) {
        if (isset($php_caller_map[$u1_caller_func])) {
            $called_by_u1_php = array_merge($called_by_u1_php, $php_caller_map[$u1_caller_func]);
        }
    }
    $analysis_results['php']['called_by_unused_level1'] = array_values(array_unique(array_intersect($called_by_u1_php, $defined_php_functions)));
    sort($analysis_results['php']['called_by_unused_level1']);

    // --- JS Analysis ---
    $js_function_bodies = []; 
    $defined_js_functions = [];
    $js_script_content_for_global_calls = "";

    preg_match_all('/<script(?![^>]*src=)[^>]*>(.*?)<\/script>/is', $file_content, $scriptBlocks);
    if (!empty($scriptBlocks[1])) {
        foreach ($scriptBlocks[1] as $script_content_block) {
            $js_script_content_for_global_calls .= $script_content_block . "\n";
            $functions_in_block = get_function_bodies_with_names($script_content_block, 'js');
            foreach($functions_in_block as $name => $body) {
                if (!isset($js_function_bodies[$name])) {
                    $js_function_bodies[$name] = $body;
                }
                if(!in_array($name, $defined_js_functions)) {
                    $defined_js_functions[] = $name;
                }
            }
        }
    }
    sort($defined_js_functions);
    $analysis_results['js']['defined'] = $defined_js_functions;

    $js_caller_map = [];
    $js_call_pattern = '/(?<!function\s)([a-zA-Z_$][a-zA-Z0-9_$]*)\s*\(/';

    foreach ($js_function_bodies as $caller_name => $body) {
        preg_match_all($js_call_pattern, $body, $calls_in_body);
        $unique_calls = array_unique($calls_in_body[1]);
        $valid_calls = array_intersect($unique_calls, $defined_js_functions);
        $js_caller_map[$caller_name] = array_values($valid_calls);
    }
    
    $js_entry_points = [];
    if(!empty($js_script_content_for_global_calls)) {
        preg_match_all($js_call_pattern, $js_script_content_for_global_calls, $all_js_calls_match);
        if (!empty($all_js_calls_match[1])) {
            $all_js_calls_in_scripts = array_unique($all_js_calls_match[1]);
            $all_js_calls_to_defined_functions = array_intersect($all_js_calls_in_scripts, $defined_js_functions);

            $js_calls_from_within_functions = [];
            foreach($js_caller_map as $callees) {
                $js_calls_from_within_functions = array_merge($js_calls_from_within_functions, $callees);
            }
            $js_calls_from_within_functions = array_unique($js_calls_from_within_functions);
            
            $global_scope_js_calls = array_diff($all_js_calls_to_defined_functions, $js_calls_from_within_functions);
            $js_entry_points = array_values($global_scope_js_calls);
        }
    }

    $html_event_handler_calls = [];
    $event_handler_pattern = '/\s(on[a-zA-Z]+)\s*=\s*["\']([^"\']+)["\']/i';
    preg_match_all($event_handler_pattern, $file_content, $event_matches, PREG_SET_ORDER);

    foreach ($event_matches as $match) {
        $handler_code = $match[2];
        preg_match_all($js_call_pattern, $handler_code, $calls_in_handler);
        if (!empty($calls_in_handler[1])) {
            foreach ($calls_in_handler[1] as $func_name_in_handler) {
                if (in_array($func_name_in_handler, $defined_js_functions)) {
                    $html_event_handler_calls[] = $func_name_in_handler;
                }
            }
        }
    }
    $js_entry_points = array_unique(array_merge($js_entry_points, $html_event_handler_calls));
    
    $js_protocol_calls = [];
    $js_protocol_pattern = '/\s(?:href|action)\s*=\s*(["\'])\s*javascript:(.*?)\1/i'; 
    preg_match_all($js_protocol_pattern, $file_content, $protocol_matches, PREG_SET_ORDER);

    foreach ($protocol_matches as $match) {
        $js_code = $match[2];
        preg_match_all($js_call_pattern, $js_code, $calls_in_js_code);
        if (!empty($calls_in_js_code[1])) {
            foreach ($calls_in_js_code[1] as $func_name_in_code) {
                if (in_array($func_name_in_code, $defined_js_functions)) {
                    $js_protocol_calls[] = $func_name_in_code;
                }
            }
        }
    }
    $js_entry_points = array_unique(array_merge($js_entry_points, $js_protocol_calls));
    $js_entry_points = array_values(array_intersect($js_entry_points, $defined_js_functions));

    $reachable_js = [];
    $queue_js = $js_entry_points;
     while(!empty($queue_js)) {
        $func = array_shift($queue_js);
        if (in_array($func, $defined_js_functions) && !in_array($func, $reachable_js)) {
            $reachable_js[] = $func;
            if (isset($js_caller_map[$func])) {
                foreach ($js_caller_map[$func] as $called_func) {
                     if (!in_array($called_func, $reachable_js)) {
                        $queue_js[] = $called_func;
                    }
                }
            }
        }
    }
    sort($reachable_js);
    $analysis_results['js']['unused_level0'] = array_values(array_diff($defined_js_functions, $reachable_js));

    $called_by_u0_js = [];
    foreach ($analysis_results['js']['unused_level0'] as $u0_func) {
        if (isset($js_caller_map[$u0_func])) {
            $called_by_u0_js = array_merge($called_by_u0_js, $js_caller_map[$u0_func]);
        }
    }
    $analysis_results['js']['called_by_unused_level0'] = array_values(array_unique(array_intersect($called_by_u0_js, $defined_js_functions)));
    sort($analysis_results['js']['called_by_unused_level0']);
    
    $called_by_u1_js = [];
    foreach ($analysis_results['js']['called_by_unused_level0'] as $u1_caller_func) {
        if (isset($js_caller_map[$u1_caller_func])) {
            $called_by_u1_js = array_merge($called_by_u1_js, $js_caller_map[$u1_caller_func]);
        }
    }
    $analysis_results['js']['called_by_unused_level1'] = array_values(array_unique(array_intersect($called_by_u1_js, $defined_js_functions)));
    sort($analysis_results['js']['called_by_unused_level1']);

    // Add caller maps to the final result
    $analysis_results['php']['caller_map'] = $php_caller_map;
    $analysis_results['js']['caller_map'] = $js_caller_map;

    return $analysis_results;
}

/**
 * Generates the comprehensive codebase index as a formatted string.
 * @param string $content The content of the file to be indexed.
 * @return string The formatted index text.
 */
function generateCodebaseIndex($content) {
    $output = "";
    $output .= "VYROX AI CODE ASSISTANT - COMPREHENSIVE CODEBASE INDEX\n" . str_repeat("=", 80) . "\n\n";
    $output .= "JSON REQUEST FORMAT:\n{\n" .
         '  "php": ["function_name"],' . "\n" . '  "php_class": ["class_name"],' . "\n" .
         '  "js": ["js_function_name"],' . "\n" .
         '  "html": [".class", "#id", "forms", "buttons", "head"],' . "\n" . '  "comments": ["Section Name"],' . "\n" .
         '  "blocks": ["php_opening", "ajax_handler"],' . "\n" .
         '  "post_handlers": ["handler_name"],' . "\n" .
         '  "get_handlers": ["handler_name"],' . "\n" .
         '  "switch_case": ["case_value"],' . "\n" .
         '  "create_tables": ["table_name"]'."\n}\n\n";
    
    $usage_analysis = analyze_function_usage($content);

    // --- PHP Function Index with Call Graph ---
    $phpFunctions = $usage_analysis['php']['defined'];
    $phpCallerMap = $usage_analysis['php']['caller_map'];
    $output .= "INDEX 1: PHP FUNCTIONS (" . count($phpFunctions) . " defined)\n".str_repeat("-",30)."\n";
    if (empty($phpFunctions)) {
        $output .= "- None found\n";
    } else {
        foreach ($phpFunctions as $func) {
            $output .= "- " . htmlspecialchars($func) . "\n";
            if (isset($phpCallerMap[$func]) && !empty($phpCallerMap[$func])) {
                $output .= "    -> calls: " . htmlspecialchars(implode(', ', $phpCallerMap[$func])) . "\n";
            }
        }
    }
    $output .= "\n";
    
    preg_match_all('/^\s*(?:abstract\s+|final\s+)*class\s+([a-zA-Z_\x7f-\xff][a-zA-Z0-9_\x7f-\xff]*)/im', $content, $classMatches);
    $phpClasses = array_unique($classMatches[1]);
    sort($phpClasses);
    $output .= "INDEX 2: PHP CLASSES (" . count($phpClasses) . " found)\n".str_repeat("-",28)."\n".(empty($phpClasses)?"- None found\n":"- ".implode("\n - ", $phpClasses)."\n")."\n";
    
    // --- JS Function Index with Call Graph ---
    $jsFunctions = $usage_analysis['js']['defined'];
    $jsCallerMap = $usage_analysis['js']['caller_map'];
    $output .= "INDEX 3: JAVASCRIPT FUNCTIONS (".count($jsFunctions)." defined)\n".str_repeat("-",35)."\n";
    if (empty($jsFunctions)) {
        $output .= "- None found\n";
    } else {
        foreach ($jsFunctions as $func) {
            $output .= "- " . htmlspecialchars($func) . "\n";
            if (isset($jsCallerMap[$func]) && !empty($jsCallerMap[$func])) {
                $output .= "    -> calls: " . htmlspecialchars(implode(', ', $jsCallerMap[$func])) . "\n";
            }
        }
    }
    $output .= "\n";

    $commentBlocks = [];
    if (preg_match('/<body[^>]*>(.*?)<\/body>/is', $content, $bodyMatch)) {
        preg_match_all('/<!--\s*(.*?)\s*-->/s', $bodyMatch[1], $commentMatches);
        $commentBlocks = array_unique(array_filter(array_map('trim', $commentMatches[1])));
        sort($commentBlocks);
    }
    $output .= "INDEX 4: HTML COMMENT BLOCKS (".count($commentBlocks)." found)\n".str_repeat("-",35)."\n".(empty($commentBlocks)?"- None found (or no body tag)\n":"- ".implode("\n - ", $commentBlocks)."\n")."\n";

    $handlers = extractHandlers($content);
    $output .= "INDEX 5: POST HANDLERS (".count($handlers['post'])." found)\n".str_repeat("-",25)."\n".(empty($handlers['post'])?"- None found\n":"- ".implode("\n - ", $handlers['post'])."\n")."\n";
    $output .= "INDEX 6: GET HANDLERS (".count($handlers['get'])." found)\n".str_repeat("-",24)."\n".(empty($handlers['get'])?"- None found\n":"- ".implode("\n - ", $handlers['get'])."\n")."\n";
    
    $caseValues = extractAllCaseValues($content);
    $output .= "INDEX 7: SWITCH CASE VALUES (".count($caseValues)." found)\n".str_repeat("-",32)."\n".(empty($caseValues)?"- None found\n":"- ".implode("\n - ", $caseValues)."\n")."\n";

    $output .= "INDEX 8: HTML ELEMENTS (SELECTORS)\n".str_repeat("-",35)."\n";
    $ids = []; $classes = [];
    preg_match_all('/id="([^"]+)"/i', $content, $idMatches);
    if(!empty($idMatches[1])) { $ids = array_unique($idMatches[1]); sort($ids); }
    $output .= "IDs (".count($ids)." found): ".(empty($ids)?"None":"#".implode(', #', $ids))."\n";
    
    preg_match_all('/class="([^"]+)"/i', $content, $classMatches);
    if(!empty($classMatches[1])) {
        $allClasses = [];
        foreach($classMatches[1] as $classList) $allClasses = array_merge($allClasses, preg_split('/\s+/', trim($classList)));
        $classes = array_unique(array_filter($allClasses)); sort($classes);
    }
    $output .= "Classes (".count($classes)." found): ".(empty($classes)?"None":".".implode(', .', $classes))."\n\n";
    
    $createTables = [];
    preg_match_all('/[\'"](\w+)[\'"]\s*=>\s*[\'"]CREATE TABLE/i', $content, $createTableMatches);
    if(!empty($createTableMatches[1])) { $createTables = array_unique($createTableMatches[1]); sort($createTables); }
    $output .= "INDEX 9: CREATE TABLE DEFINITIONS (".count($createTables)." found)\n".str_repeat("-",40)."\n".(empty($createTables)?"- None found\n":"- ".implode("\n - ", $createTables)."\n")."\n";

    $output .= "INDEX 10: UNUSED PHP FUNCTION ANALYSIS\n".str_repeat("-",40)."\n";
    $unused_php_l0 = $usage_analysis['php']['unused_level0'];
    $output .= "Level 0 (Potentially Unused/Never Called Directly): ".count($unused_php_l0)." found\n".(empty($unused_php_l0)?"- None\n":"- ".implode("\n - ", $unused_php_l0)."\n")."\n";
    $called_by_unused_php_l0 = array_diff($usage_analysis['php']['called_by_unused_level0'], $unused_php_l0);
    $output .= "Level 1 (Called ONLY by Level 0 functions): ".count($called_by_unused_php_l0)." found\n".(empty($called_by_unused_php_l0)?"- None\n":"- ".implode("\n - ", $called_by_unused_php_l0)."\n")."\n";
    $called_by_unused_php_l1 = array_diff($usage_analysis['php']['called_by_unused_level1'], $unused_php_l0, $called_by_unused_php_l0);
    $output .= "Level 2 (Called ONLY by Level 1 functions): ".count($called_by_unused_php_l1)." found\n".(empty($called_by_unused_php_l1)?"- None\n":"- ".implode("\n - ", $called_by_unused_php_l1)."\n")."\n";

    $output .= "INDEX 11: UNUSED JAVASCRIPT FUNCTION ANALYSIS\n".str_repeat("-",46)."\n";
    $unused_js_l0 = $usage_analysis['js']['unused_level0'];
    $output .= "Level 0 (Potentially Unused/Never Called Directly - check dynamic calls): ".count($unused_js_l0)." found\n".(empty($unused_js_l0)?"- None\n":"- ".implode("\n - ", $unused_js_l0)."\n")."\n";
    $called_by_unused_js_l0 = array_diff($usage_analysis['js']['called_by_unused_level0'], $unused_js_l0);
    $output .= "Level 1 (Called ONLY by Level 0 functions): ".count($called_by_unused_js_l0)." found\n".(empty($called_by_unused_js_l0)?"- None\n":"- ".implode("\n - ", $called_by_unused_js_l0)."\n")."\n";
    $called_by_unused_js_l1 = array_diff($usage_analysis['js']['called_by_unused_level1'], $unused_js_l0, $called_by_unused_js_l0);
    $output .= "Level 2 (Called ONLY by Level 1 functions): ".count($called_by_unused_js_l1)." found\n".(empty($called_by_unused_js_l1)?"- None\n":"- ".implode("\n - ", $called_by_unused_js_l1)."\n")."\n";

    $output .= str_repeat("=", 80) . "\nEND OF INDEX\nPLEASE REQUEST THE CODEBLOCKS YOU NEED UNTIL YOU HAVE ENOUGH INFORMATION TO SEE THE COMPLETE PICTURE AND ABLE TO PINPOINT THE PROBLEMATIC CODEBLOCK\nCHECK ALL THE RELATED FUNCTIONS AND FIX WITH MOST MINIMUM CHANGE\nMODIFY EXISTING FUNCTION WITH THE MOST MINIMUM CHANGE\nDO NOT SIMPLY CREATE NEW FUNCTION";
    
    return $output;
}

/**
 * Returns the comprehensive API usage instructions as a string.
 * @return string The formatted instructions text.
 */
function getApiInstructions() {
    return <<<DOCS
=================================================================
COMPREHENSIVE API USAGE INSTRUCTIONS
=================================================================

This tool provides a powerful API for programmatic access. There are
three main commands you can issue, controlled by URL parameters.

All requests should be made to: [path_to_this_file]/tool.php?api=1

-----------------------------------------------------------------
COMMAND 1: EXTRACTING CODE BLOCKS
-----------------------------------------------------------------
This is the primary function of the API. It allows you to request
specific pieces of code from the target file.

- ENDPOINT: [path_to_this_file]/tool.php?api=1
- You can also specify the context lines: ...?api=1&context=25

- METHOD: POST

- HEADERS:
  - Content-Type: application/json

- BODY:
  - The body of the POST request must be a raw JSON string.
  - The keys correspond to the type of code to extract (e.g., "php", "js", "html").
  - The values are arrays of strings, where each string is the name or
    selector of the item to extract.

- SUCCESS RESPONSE (200 OK):
  - A JSON object containing the extracted code.
  - Format: {"extractedCode": "..."}

- ERROR RESPONSE:
  - A JSON object containing an error description.
  - Format: {"error": "..."}

- COMPREHENSIVE cURL EXAMPLE:
  This command demonstrates a request for multiple types of code blocks
  simultaneously.

  curl -X POST \\
    'http://localhost/path/to/tool.php?api=1&context=25' \\
    -H 'Content-Type: application/json' \\
    -d '{
      "php": ["process_data", "validate_user_input"],
      "php_class": ["DatabaseManager"],
      "js": ["updateUI"],
      "html": ["#main-content", ".btn-submit", "forms"],
      "comments": ["User Profile Section"],
      "blocks": ["ajax_handler"],
      "post_handlers": ["save_user_profile"],
      "get_handlers": ["fetch_user_data"],
      "switch_case": ["delete_record"],
      "create_tables": ["users"]
    }'

-----------------------------------------------------------------
COMMAND 2: REQUESTING THE CODEBASE INDEX
-----------------------------------------------------------------
To retrieve the full, pre-generated codebase index as a string, make a
GET request to the API endpoint with the 'index' parameter. This does
not require a POST body.

- ENDPOINT: [path_to_this_file]/tool.php?api=1&index=1

- METHOD: GET

- SUCCESS RESPONSE (200 OK):
  - A JSON object containing the complete index text.
  - Format: {"codebaseIndex": "AI CODE ASSISTANT - COMPREHENSIVE CODEBASE INDEX\\n..."}

- cURL EXAMPLE:
  curl -X GET 'http://localhost/path/to/tool.php?api=1&index=1'

-----------------------------------------------------------------
COMMAND 3: REQUESTING API INSTRUCTIONS (THIS DOCUMENT)
-----------------------------------------------------------------
To retrieve this full set of API instructions programmatically.

- ENDPOINT: [path_to_this_file]/tool.php?api=1&instructions=1

- METHOD: GET

- SUCCESS RESPONSE (200 OK):
  - A JSON object containing this documentation.
  - Format: {"apiInstructions": "================================...etc"}

- cURL EXAMPLE:
  curl -X GET 'http://localhost/path/to/tool.php?api=1&instructions=1'
DOCS;
}


// =================================================================
// AJAX HANDLER & MAIN APPLICATION LOGIC
// =================================================================

// This block now handles both frontend AJAX requests (?ajax=1) and external API requests (?api=1)
if ((isset($_GET['ajax']) && $_GET['ajax'] == '1') || (isset($_GET['api']) && $_GET['api'] == '1')) {
    header('Content-Type: application/json');
    $targetFile = 'index.php';
    if (!file_exists($targetFile)) {
        echo json_encode(['error' => "Target file '$targetFile' not found."]);
        exit;
    }
    $content = file_get_contents($targetFile);
    if ($content === false) {
        echo json_encode(['error' => "Could not read target file '$targetFile'."]);
        exit;
    }

    // New feature: Return API instructions if requested
    if (isset($_GET['api']) && isset($_GET['instructions']) && $_GET['instructions'] == '1') {
        echo json_encode(['apiInstructions' => getApiInstructions()]);
        exit;
    }

    // Return the codebase index if requested via the API
    if (isset($_GET['api']) && isset($_GET['index']) && $_GET['index'] == '1') {
        $indexContent = generateCodebaseIndex($content);
        echo json_encode(['codebaseIndex' => $indexContent]);
        exit;
    }

    $input = json_decode(file_get_contents('php://input'), true);
    if ($input === null && json_last_error() !== JSON_ERROR_NONE) {
        echo json_encode(['error' => 'Invalid JSON input: ' . json_last_error_msg()]);
        exit;
    }
    if (empty($input)) {
        $input = [];
    }

    $contextLines = isset($_GET['context']) ? (int)$_GET['context'] : 50;
    $contextLines = max(10, min(200, $contextLines));
    $result = "AI CODE EXTRACTION RESULTS\n" . str_repeat("=", 80) . "\n\n";

    $extractionMap = [
        'php' => 'extractPhpFunction', 'php_class' => 'extractPhpClass', 'js' => 'extractJsFunction',
        'html' => 'extractHtmlContent', 'comments' => 'extractCommentBlock',
        'blocks' => 'extractCodeBlock', 'post_handlers' => 'extractHandler',
        'get_handlers' => 'extractHandler', 'create_tables' => 'extractTableDefinition',
        'switch_case' => 'extractSwitchCase',
    ];

    foreach ($extractionMap as $key => $funcName) {
        if (isset($input[$key]) && is_array($input[$key])) {
            $title = strtoupper(str_replace('_', ' ', $key));
            $result .= "$title:\n" . str_repeat("-", strlen($title) + 1) . "\n";
            $anyFoundInCategory = false;

            foreach ($input[$key] as $item) {
                if (!is_string($item) && !is_numeric($item)) continue;

                $args = [$content, (string)$item, $contextLines];
                if ($key === 'post_handlers') $args = [$content, (string)$item, 'post', $contextLines];
                if ($key === 'get_handlers') $args = [$content, (string)$item, 'get', $contextLines];
                
                $extractedBlocks = call_user_func_array($funcName, $args);
                
                if (!empty($extractedBlocks) && is_array($extractedBlocks)) {
                    $anyFoundInCategory = true;
                    foreach ($extractedBlocks as $block) {
                        $result .= $block . "\n" . str_repeat("-", 40) . "\n\n";
                    }
                }
            }
            if (!$anyFoundInCategory) {
                 $result .= "// No items found or extracted for this category with the given names.\n\n";
            }
        }
    }
    
    $duplicates = detectDuplicateFunctions($result);
    if (!empty($duplicates['php']) || !empty($duplicates['js'])) {
        $result .= "\n" . str_repeat("!", 80) . "\nCRITICAL: DUPLICATE FUNCTION DEFINITIONS DETECTED\n" . str_repeat("!", 80) . "\n\n";
        $result .= "AI INSTRUCTIONS FOR FIXING DUPLICATES:\n";
        $result .= "1. ANALYZE all versions of each duplicate function provided in this output.\n";
        $result .= "2. MERGE all unique logic from every version into a single, consolidated function.\n";
        $result .= "3. DO NOT simply delete one version. This will cause functionality to be lost.\n";
        $result .= "4. The final, merged function must handle all original use cases and parameters gracefully.\n";
        $result .= "5. UPDATE all places in the code that call the old functions to use the new, single, merged function.\n\n";
        if (!empty($duplicates['php'])) $result .= "DUPLICATE PHP FUNCTIONS FOUND: " . implode(', ', $duplicates['php']) . "\n";
        if (!empty($duplicates['js'])) $result .= "DUPLICATE JAVASCRIPT FUNCTIONS FOUND: " . implode(', ', $duplicates['js']) . "\n";
        $result .= str_repeat("!", 80) . "\n\n";
    }

    $result .= "EXTRACTION COMPLETE\nTotal characters extracted: " . strlen($result) . "\n" . str_repeat("=", 80);
    echo json_encode(['extractedCode' => $result]);
    exit;
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>AI Code Assistant</title>
<style>
    :root {
        --primary-color: #007cba; --primary-hover: #005a8b; --border-color: #ddd;
        --bg-light: #f9f9f9; --bg-info: #f0f8ff; --success-color: #28a745;
        --success-bg: #d4edda; --error-color: #dc3545; --error-bg: #f8d7da;
    }
    body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; margin: 0; padding: 20px; background-color: #f4f4f9; color: #333; }
    h2 { text-align: center; color: var(--primary-color); margin-bottom: 20px; }
    .container { display: flex; gap: 20px; height: calc(100vh - 80px); max-width: 1800px; margin: 0 auto; }
    .column { display: flex; flex-direction: column; background: #fff; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.05); overflow: hidden; }
    .left-column { width: 60%; } .right-column { width: 40%; }
    .column-header { padding: 10px 15px; border-bottom: 1px solid var(--border-color); background-color: var(--bg-light); font-weight: bold; display: flex; justify-content: space-between; align-items: center; }
    .content { flex: 1; padding: 15px; overflow-y: auto; position: relative; }
    .left-column .top-content { flex: 0 0 auto; border-bottom: 1px solid var(--border-color); }
    textarea { width: calc(100% - 22px); height: 120px; padding: 10px; border: 1px solid var(--border-color); border-radius: 4px; resize: vertical; font-family: "Fira Code", "Courier New", monospace; font-size: 14px; transition: border-color 0.2s, background-color 0.2s; }
    textarea:focus { outline: none; border-color: var(--primary-color); }
    textarea.error { border-color: var(--error-color); background-color: var(--error-bg); }
    .json-error { color: var(--error-color); font-size: 12px; margin-top: 5px; display: none; }
    .controls { display: flex; gap: 10px; align-items: center; margin-bottom: 10px; }
    button { background: var(--primary-color); color: white; padding: 10px 20px; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; transition: background-color 0.2s, transform 0.1s; }
    button:hover { background: var(--primary-hover); }
    button:active { transform: scale(0.98); }
    .copy-btn { padding: 5px 10px; font-size: 12px; background: #6c757d; }
    .copy-btn.copied { background: var(--success-color) !important; }
    pre { white-space: pre-wrap; word-wrap: break-word; margin: 0; font-size: 12px; font-family: "Fira Code", "Courier New", monospace; }
    .line-count { background: rgba(0,0,0,0.05); padding: 3px 8px; border-radius: 10px; font-size: 11px; color: #666; font-weight: normal; }
    .context-control { margin-right: auto; font-size: 14px; }
    .context-control input { width: 60px; padding: 5px; border-radius: 4px; border: 1px solid var(--border-color); text-align: center; }
    .loader-overlay { position: absolute; top: 0; left: 0; right: 0; bottom: 0; background: rgba(255, 255, 255, 0.8); display: none; justify-content: center; align-items: center; z-index: 10; }
    .loader { border: 5px solid #f3f3f3; border-top: 5px solid var(--primary-color); border-radius: 50%; width: 50px; height: 50px; animation: spin 1s linear infinite; }
    @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
</style>
</head>
<body>
<h2>AI Code Assistant</h2>
<div class="container">
   <div class="column left-column">
       <div class="top-content content">
           <div class="controls">
               <div class="context-control" title="The number of extra lines to show before and after the extracted code block for better analysis.">
                   <label for="contextLines">Context (before/after):</label>
                   <input type="number" id="contextLines" value="50" min="10" max="200">
               </div>
               <button id="extractBtn">Extract Code (Ctrl+Enter)</button>
               <button id="clearBtn" class="copy-btn">Clear</button>
           </div>
           <textarea id="aiRequest" placeholder="Paste your JSON request here... (e.g., {"php": ["myFunction"]})"></textarea>
           <div id="jsonError" class="json-error"></div>
       </div>
       <div class="column-header">
           <span>Extracted Code</span>
           <span class="line-count" id="leftLineCount">0 lines</span>
           <button class="copy-btn" id="leftCopyBtn">Copy</button>
       </div>
       <div class="content" id="leftContainer">
           <div class="loader-overlay" id="loader"><div class="loader"></div></div>
           <pre id="result">Welcome! The codebase index on the right is now on your clipboard. Paste a JSON request above to begin.</pre>
       </div>
   </div>
   <div class="column right-column">
       <div class="column-header">
           <span>Codebase Index</span>
           <span class="line-count" id="rightLineCount"><?php
                $indexPath = 'index.php';
                $indexContentForCount = '';
                if (file_exists($indexPath)) {
                    $indexContentForCount = file_get_contents($indexPath);
                }
                echo countTotalLines($indexContentForCount);
           ?> lines</span>
           <button class="copy-btn" id="rightCopyBtn">Copy</button>
       </div>
       <div class="content" id="rightContainer">
           <pre id="rightContent"><?php
               $indexPath = 'index.php';
               if (file_exists($indexPath)) {
                   $content = file_get_contents($indexPath);
                   if ($content === false) {
                       echo "ERROR: Could not read $indexPath.";
                   } else {
                       // Use the refactored function to generate and echo the index
                       echo generateCodebaseIndex($content);
                   }
               } else { echo "ERROR: $indexPath file not found."; }
           ?></pre>
       </div>
   </div>
</div>
<script>
document.addEventListener('DOMContentLoaded', () => {
    const aiRequest = document.getElementById('aiRequest');
    const resultPre = document.getElementById('result');
    const extractBtn = document.getElementById('extractBtn');
    const clearBtn = document.getElementById('clearBtn');
    const leftCopyBtn = document.getElementById('leftCopyBtn');
    const rightCopyBtn = document.getElementById('rightCopyBtn');
    const loader = document.getElementById('loader');
    const jsonError = document.getElementById('jsonError');
    const rightContentEl = document.getElementById('rightContent');

    const updateLineCounts = () => {
        const leftContent = resultPre.textContent || '';
        document.getElementById('leftLineCount').textContent = `${leftContent.split('\n').length} lines`;
        
        if (rightContentEl) {
            const rightContent = rightContentEl.textContent || '';
            document.getElementById('rightLineCount').textContent = `${rightContent.split('\n').length} lines`;
        }
    };

    const copyToClipboard = (text, btn) => {
        if (typeof text !== 'string' || !text.trim() || !navigator.clipboard) {
            console.warn('Copy to clipboard: No text or clipboard API unavailable.');
            if (btn) {
                const originalText = btn.textContent;
                btn.textContent = 'No text!';
                setTimeout(() => { btn.textContent = originalText; }, 1500);
            }
            return;
        }
        navigator.clipboard.writeText(text).then(() => {
            if (btn) {
                const originalText = btn.textContent;
                btn.textContent = 'Copied!';
                btn.classList.add('copied');
                setTimeout(() => { btn.textContent = originalText; btn.classList.remove('copied'); }, 1500);
            }
        }).catch(err => {
            console.error('Failed to copy text: ', err);
            if (btn) {
                const originalText = btn.textContent;
                btn.textContent = 'Copy Fail!';
                setTimeout(() => { btn.textContent = originalText; }, 1500);
            }
        });
    };

    const showLoader = (show) => { loader.style.display = show ? 'flex' : 'none'; };

    const handleExtraction = () => {
        const requestText = aiRequest.value.trim();
        if (!requestText) {
            jsonError.textContent = 'Please enter a JSON request.';
            jsonError.style.display = 'block';
            aiRequest.classList.add('error');
            return;
        }
        let requestData;
        try {
            requestData = JSON.parse(requestText);
            aiRequest.classList.remove('error');
            jsonError.style.display = 'none';
        } catch (e) {
            aiRequest.classList.add('error');
            jsonError.textContent = `JSON Parse Error: ${e.message}`;
            jsonError.style.display = 'block';
            return;
        }
        
        showLoader(true);
        const contextLines = document.getElementById('contextLines').value;
        fetch(`?ajax=1&context=${contextLines}`, {
            method: 'POST',
            headers: {'Content-Type': 'application/json', 'Accept': 'application/json'},
            body: JSON.stringify(requestData)
        })
        .then(response => {
            if (!response.ok) {
                return response.text().then(text => { throw new Error(`Server Error: ${response.status} ${response.statusText} - ${text}`); });
            }
            return response.json();
        })
        .then(data => {
            if (data.error) {
                resultPre.textContent = `SERVER ERROR: ${data.error}`;
            } else if (data.extractedCode !== undefined) {
                resultPre.textContent = data.extractedCode;
                copyToClipboard(data.extractedCode, leftCopyBtn);
            } else {
                resultPre.textContent = 'SERVER RESPONSE: Received empty or unexpected data.';
            }
        })
        .catch(error => {
            resultPre.textContent = `FETCH/PROCESSING ERROR: ${error.message}`;
            console.error("Extraction error:", error);
        })
        .finally(() => {
            showLoader(false);
            updateLineCounts();
        });
    };
    
    updateLineCounts();
    if (rightContentEl && rightContentEl.textContent.trim()) {
        copyToClipboard(rightContentEl.textContent, rightCopyBtn);
    }

    extractBtn.addEventListener('click', handleExtraction);
    clearBtn.addEventListener('click', () => {
        aiRequest.value = '';
        aiRequest.classList.remove('error');
        jsonError.style.display = 'none';
        resultPre.textContent = 'Extraction results will appear here.';
        updateLineCounts();
    });
    leftCopyBtn.addEventListener('click', () => copyToClipboard(resultPre.textContent, leftCopyBtn));
    rightCopyBtn.addEventListener('click', () => {
        if (rightContentEl) copyToClipboard(rightContentEl.textContent, rightCopyBtn);
    });

    aiRequest.addEventListener('paste', (event) => {
        setTimeout(handleExtraction, 50);
    });
    aiRequest.addEventListener('input', () => {
        if (aiRequest.classList.contains('error')) {
            try {
                if (aiRequest.value.trim()) JSON.parse(aiRequest.value);
                aiRequest.classList.remove('error');
                jsonError.style.display = 'none';
            } catch (e) { /* Still invalid JSON, error remains */ }
        }
    });
    document.addEventListener('keydown', (e) => {
        if ((e.ctrlKey || e.metaKey) && e.key === 'Enter') {
            e.preventDefault();
            handleExtraction();
        }
    });
});
</script>
</body>
</html>
<?php
// The final documentation block is now generated dynamically to ensure it's always in sync with the API.
echo "/*\n" . getApiInstructions() . "\n*/\n";
?>
