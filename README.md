# OOPS-PROJECT1
nner is closed
     */
    public Scanner skip(Pattern pattern) {
        ensureOpen();
        if (pattern == null)
            throw new NullPointerException();
        clearCaches();
        modCount++;

        // Search for the pattern
        while (true) {
            if (matchPatternInBuffer(pattern)) {
                matchValid = true;
                position = matcher.end();
                return this;
            }
            if (needInput)
                readInput();
            else
                throw new NoSuchElementException();
        }
    }

    /**
     * Skips input that matches a pattern constructed from the specified
     * string.
     *
     * <p> An invocation of this method of the form {@code skip(pattern)}
     * behaves in exactly the same way as the invocation
     * {@code skip(Pattern.compile(pattern))}.
     *
     * @param pattern a string specifying the pattern to skip over
     * @return this scanner
     * @throws IllegalStateException if this scanner is closed
     */
    public Scanner skip(String pattern) {
        return skip(patternCache.forName(pattern));
    }

    // Convenience methods for scanning primitives

    /**
     * Returns true if the next token in this scanner's input can be
     * interpreted as a boolean value using a case insensitive pattern
     * created from the string "true|false".  The scanner does not
     * advance past the input that matched.
     *
     * @return true if and only if this scanner's next token is a valid
     *         boolean value
     * @throws IllegalStateException if this scanner is closed
     */
    public boolean hasNextBoolean()  {
        return hasNext(boolPattern());
    }

    /**
     * Scans the next token of the input into a boolean value and returns
     * that value. This method will throw {@code InputMismatchException}
     * if the next token cannot be translated into a valid boolean value.
     * If the match is successful, the scanner advances past the input that
     * matched.
     *
     * @return the boolean scanned from the input
     * @throws InputMismatchException if the next token is not a valid boolean
     * @throws NoSuchElementException if input is exhausted
     * @throws IllegalStateException if this scanner is closed
     */
    public boolean nextBoolean()  {
        clearCaches();
        return Boolean.parseBoolean(next(boolPattern()));
    }

    /**
     * Returns true if the next token in this scanner's input can be
     * interpreted as a byte value in the default radix using the
     * {@link #nextByte} method. The scanner does not advance past any input.
     *
     * @return true if and only if this scanner's next token is a valid
     *         byte value
     * @throws IllegalStateException if this scanner is closed
     */
    public boolean hasNextByte() {
        return hasNextByte(defaultRadix);
    }

    /**
     * Returns true if the next token in this scanner's input can be
     * interpreted as a byte value in the specified radix using the
     * {@link #nextByte} method. The scanner does not advance past any input.
     *
     * <p>If the radix is less than {@link Character#MIN_RADIX Character.MIN_RADIX}
     * or greater than {@link Character#MAX_RADIX Character.MAX_RADIX}, then an
     * {@code IllegalArgumentException} is thrown.
     *
     * @param radix the radix used to interpret the token as a byte value
     * @return true if and only if this scanner's next token is a valid
     *         byte value
     * @throws IllegalStateException if this scanner is closed
     * @throws IllegalArgumentException if the radix is out of range
     */
    public boolean hasNextByte(int radix) {
        setRadix(radix);
        boolean result = hasNext(integerPattern());
        if (result) { // Cache it
            try {
                String s = (matcher.group(SIMPLE_GROUP_INDEX) == null) ?
                    processIntegerToken(hasNextResult) :
                    hasNextResult;
                typeCache = Byte.parseByte(s, radix);
            } catch (NumberFormatException nfe) {
                result = false;
            }
        }
        return result;
    }

    /**
     * Scans the next token of the input as a {@code byte}.
     *
     * <p> An invocation of this method of the form
     * {@code nextByte()} behaves in exactly the same way as the
     * invocation {@code nextByte(radix)}, where {@code radix}
     * is the default radix of this scanner.
     *
     * @return the {@code byte} scanned from the input
     * @throws InputMismatchException
     *         if the next token does not match the <i>Integer</i>
     *         regular expression, or is out of range
     * @throws NoSuchElementException if input is exhausted
     * @throws IllegalStateException if this scanner is closed
     */
    public byte nextByte() {
         return nextByte(defaultRadix);
    }

    /**
     * Scans the next token of the input as a {@code byte}.
     * This method will throw {@code InputMismatchException}
     * if the next token cannot be translated into a valid byte value as
     * described below. If the translation is successful, the scanner advances
     * past the input that matched.
     *
     * <p> If the next token matches the <a
     * href="#Integer-regex"><i>Integer</i></a> regular expression defined
     * above then the token is converted into a {@code byte} value as if by
     * removing all locale specific prefixes, group separators, and locale
     * specific suffixes, then mapping non-ASCII digits into ASCII
     * digits via {@link Character#digit Character.digit}, prepending a
     * negative sign (-) if the locale specific negative prefixes and suffixes
     * were present, and passing the resulting string to
     * {@link Byte#parseByte(String, int) Byte.parseByte} with the
     * specified radix.
     *
     * <p>If the radix is less than {@link Character#MIN_RADIX Character.MIN_RADIX}
     * or greater than {@link Character#MAX_RADIX Character.MAX_RADIX}, then an
     * {@code IllegalArgumentException} is thrown.
     *
     * @param radix the radix used to interpret the token as a byte value
     * @return the {@code byte} scanned from the input
     * @throws InputMismatchException
     *         if the next token does not match the <i>Integer</i>
     *         regular expression, or is out of range
     * @throws NoSuchElementException if input is exhausted
     * @throws IllegalStateException if this scanner is closed
     * @throws IllegalArgumentException if the radix is out of range
     */
    public byte nextByte(int radix) {
        // Check cached result
        if ((typeCache != null) && (typeCache instanceof Byte)
            && this.radix == radix) {
            byte val = ((Byte)typeCache).byteValue();
            useTypeCache();
            return val;
        }
        setRadix(radix);
        clearCaches();
        // Search for next byte
        try {
            String s = next(integerPattern());
            if (matcher.group(SIMPLE_GROUP_INDEX) == null)
                s = processIntegerToken(s);
            return Byte.parseByte(s, radix);
        } catch (NumberFormatException nfe) {
            position = matcher.start(); // don't skip bad token
            throw new InputMismatchException(nfe.getMessage());
        }
    }

    /**
     * Returns true if the next token in this scanner's input can be
     * interpreted as a short value in the default radix using the
     * {@link #nextShort} method. The scanner does not advance past any input.
     *
     * @return true if and only if this scanner's next token is a valid
     *         short value in the default radix
     * @throws IllegalStateException if this scanner is closed
     */
    public boolean hasNextShort() {
        return hasNextShort(defaultRadix);
    }

    /**
     * Returns true if the next token in this scanner's input can be
     * interpreted as a short value in the specified radix using the
     * {@link #nextShort} method. The scanner does not advance past any input.
     *
     * <p>If the radix is less than {@link Character#MIN_RADIX Character.MIN_RADIX}
     * or greater than {@link Character#MAX_RADIX Character.MAX_RADIX}, then an
     * {@code IllegalArgumentException} is thrown.
     *
     * @param radix the radix used to interpret the token as a short value
     * @return true if and only if this scanner's next token is a valid
     *         short value in the specified radix
     * @throws IllegalStateException if this scanner is closed
     * @throws IllegalArgumentException if the radix is out of range
     */
    public boolean hasNextShort(int radix) {
        setRadix(radix);
        boolean result = hasNext(integerPattern());
        if (result) { // Cache it
            try {
                String s = (matcher.group(SIMPLE_GROUP_INDEX) == null) ?
                    processIntegerToken(hasNextResult) :
                    hasNextResult;
                typeCache = Short.parseShort(s, radix);
            } catch (NumberFormatException nfe) {
                result = false;
            }
        }
        return result;
    }

    /**
     * Scans the next token of the input as a {@code short}.
     *
     * <p> An invocation of this method of the form
     * {@code nextShort()} behaves in exactly the same way as the
     * invocation {@link #nextShort(int) nextShort(radix)}, where {@code radix}
     * is the default radix of this scanner.
     *
     * @return the {@code short} scanned from the input
     * @throws InputMismatchException
     *         if the next token does not match the <i>Integer</i>
     *         regular expression, or is out of range
     * @throws NoSuchElementException if input is exhausted
     * @throws IllegalStateException if this scanner is closed
     */
    public short nextShort() {
        return nextShort(defaultRadix);
    }

    /**
     * Scans the next token of the input as a {@code short}.
     * This method will throw {@code InputMismatchException}
     * if the next token cannot be translated into a valid short value as
     * described below. If the translation is successful, the scanner advances
     * past the input that matched.
     *
     * <p> If the next token matches the <a
     * href="#Integer-regex"><i>Integer</i></a> regular expression defined
     * above then the token is converted into a {@code short} value as if by
     * removing all locale specific prefixes, group separators, and locale
     * specific suffixes, then mapping non-ASCII digits into ASCII
     * digits via {@link Character#digit Character.digit}, prepending a
     * negative sign (-) if the locale specific negative prefixes and suffixes
     * were present, and passing the resulting string to
     * {@link Short#parseShort(String, int) Short.parseShort} with the
     * specified radix.
     *
     * <p>If the radix is less than {@link Character#MIN_RADIX Character.MIN_RADIX}
     * or greater than {@link Character#MAX_RADIX Character.MAX_RADIX}, then an
     * {@code IllegalArgumentException} is thrown.
     *
     * @param radix the radix used to interpret the token as a short value
     * @return the {@code short} scanned from the input
     * @throws InputMismatchException
     *         if the next token does not match the <i>Integer</i>
     *         regular expression, or is out of range
     * @throws NoSuchElementException if input is exhausted
     * @throws IllegalStateException if this scanner is closed
     * @throws IllegalArgumentException if the radix is out of range
     */
    public short nextShort(int radix) {
        // Check cached result
        if ((typeCache != null) && (typeCache instanceof Short)
            && this.radix == radix) {
            short val = ((Short)typeCache).shortValue();
            useTypeCache();
            return val;
        }
        setRadix(radix);
        clearCaches();
        // Search for next short
        try {
            String s = next(integerPattern());
            if (matcher.group(SIMPLE_GROUP_INDEX) == null)
                s = processIntegerToken(s);
            return Short.parseShort(s, radix);
        } catch (NumberFormatException nfe) {
            position = matcher.start(); // don't skip bad token
            throw new InputMismatchException(nfe.getMessage());
        }
    }

    /**
     * Returns true if the next token in this scanner's input can be
     * interpreted as an int value in the default radix using the
     * {@link #nextInt} method. The scanner does not advance past any input.
     *
     * @return true if and only if this scanner's next token is a valid
     *         int value
     * @throws IllegalStateException if this scanner is closed
     */
    public boolean hasNextInt() {
        return hasNextInt(defaultRadix);
    }

    /**
     * Returns true if the next token in this scanner's input can be
     * interpreted as an int value in the specified radix using the
     * {@link #nextInt} method. The scanner does not advance past any input.
     *
     * <p>If the radix is less than {@link Character#MIN_RADIX Character.MIN_RADIX}
     * or greater than {@link Character#MAX_RADIX Character.MAX_RADIX}, then an
     * {@code IllegalArgumentException} is thrown.
     *
     * @param radix the radix used to interpret the token as an int value
     * @return true if and only if this scanner's next token is a valid
     *         int value
     * @throws IllegalStateException if this scanner is closed
     * @throws IllegalArgumentException if the radix is out of range
     */
    public boolean hasNextInt(int radix) {
        setRadix(radix);
        boolean result = hasNext(integerPattern());
        if (result) { // Cache it
            try {
                String s = (matcher.group(SIMPLE_GROUP_INDEX) == null) ?
                    processIntegerToken(hasNextResult) :
                    hasNextResult;
                typeCache = Integer.parseInt(s, radix);
            } catch (NumberFormatException nfe) {
                result = false;
            }
        }
        return result;
    }

    /**
     * The integer token must be stripped of prefixes, group separators,
     * and suffixes, non ascii digits must be converted into ascii digits
     * before parse will accept it.
     */
    private String processIntegerToken(String token) {
        String result = token.replaceAll(""+groupSeparator, "");
        boolean isNegative = false;
        int preLen = negativePrefix.length();
        if ((preLen > 0) && result.startsWith(negativePrefix)) {
            isNegative = true;
            result = result.substring(preLen);
        }
        int sufLen = negativeSuffix.length();
        if ((sufLen > 0) && result.endsWith(negativeSuffix)) {
            isNegative = true;
            result = result.substring(result.length() - sufLen,
                                      result.length());
        }
        if (isNegative)
            result = "-" + result;
        return result;
    }

    /**
     * Scans the next token of the input as an {@code int}.
     *
     * <p> An invocation of this method of the form
     * {@code nextInt()} behaves in exactly the same way as the
     * invocation {@code nextInt(radix)}, where {@code radix}
     * is the default radix of this scanner.
     *
     * @return the {@code int} scanned from the input
     * @throws InputMismatchException
     *         if the next token does not match the <i>Integer</i>
     *         regular expression, or is out of range
     * @throws NoSuchElementException if input is exhausted
     * @throws IllegalStateException if this scanner is closed
     */
    public int nextInt() {
        return nextInt(defaultRadix);
    }

    /**
     * Scans the next token of the input as an {@code int}.
     * This method will throw {@code InputMismatchException}
     * if the next token cannot be translated into a valid int value as
     * described below. If the translation is successful, the scanner advances
     * past the input that matched.
     *
     * <p> If the next token matches the <a
     * href="#Integer-regex"><i>Integer</i></a> regular expression defined
     * above then the token is converted into an {@code int} value as if by
     * removing all locale specific prefixes, group separators, and locale
     * specific suffixes, then mapping non-ASCII digits into ASCII
     * digits via {@link Character#digit Character.digit}, prepending a
     * negative sign (-) if the locale specific negative prefixes and suffixes
     * were present, and passing the resulting string to
     * {@link Integer#parseInt(String, int) Integer.parseInt} with the
     * specified radix.
     *
     * <p>If the radix is less than {@link Character#MIN_RADIX Character.MIN_RADIX}
     * or greater than {@link Character#MAX_RADIX Character.MAX_RADIX}, then an
     * {@code IllegalArgumentException} is thrown.
     *
     * @param radix the radix used to interpret the token as an int value
     * @return the {@code int} scanned from the input
     * @throws InputMismatchException
     *         if the next token does not match the <i>Integer</i>
     *         regular expression, or is out of range
     * @throws NoSuchElementException if input is exhausted
     * @throws IllegalStateException if this scanner is closed
     * @throws IllegalArgumentException if the radix is out of range
     */
    public int nextInt(int radix) {
        // Check cached result
        if ((typeCache != null) && (typeCache instanceof Integer)
            && this.radix == radix) {
            int val = ((Integer)typeCache).intValue();
            useTypeCache();
            return val;
        }
        setRadix(radix);
        clearCaches();
        // Search for next int
        try {
            String s = next(integerPattern());
            if (matcher.group(SIMPLE_GROUP_INDEX) == null)
                s = processIntegerToken(s);
            return Integer.parseInt(s, radix);
        } catch (NumberFormatException nfe) {
            position = matcher.start(); // don't skip bad token
            throw new InputMismatchException(nfe.getMessage());
        }
    }

    /**
     * Returns true if the next token in this scanner's input can be
     * interpreted as a long value in the default radix using the
     * {@link #nextLong} method. The scanner does not advance past any input.
     *
     * @return true if and only if this scanner's next token is a valid
     *         long value
     * @throws IllegalStateException if this scanner is closed
     */
    public boolean hasNextLong() {
        return hasNextLong(defaultRadix);
    }

    /**
     * Returns true if the next token in this scanner's input can be
     * interpreted as a long value in the specified radix using the
     * {@link #nextLong} method. The scanner does not advance past any input.
     *
     * <p>If the radix is less than {@link Character#MIN_RADIX Character.MIN_RADIX}
     * or greater than {@link Character#MAX_RADIX Character.MAX_RADIX}, then an
     * {@code IllegalArgumentException} is thrown.
     *
     * @param radix the radix used to interpret the token as a long value
     * @return true if and only if this scanner's next token is a valid
     *         long value
     * @throws IllegalStateException if this scanner is closed
     * @throws IllegalArgumentException if the radix is out of range
     */
    public boolean hasNextLong(int radix) {
        setRadix(radix);
        boolean result = hasNext(integerPattern());
        if (result) { // Cache it
            try {
                String s = (matcher.group(SIMPLE_GROUP_INDEX) == null) ?
                    processIntegerToken(hasNextResult) :
                    hasNextResult;
                typeCache = Long.parseLong(s, radix);
            } catch (NumberFormatException nfe) {
                result = false;
            }
        }
        return result;
    }

    /**
     * Scans the next token of the input as a {@code long}.
     *
     * <p> An invocation of this method of the form
     * {@code nextLong()} behaves in exactly the same way as the
     * invocation {@code nextLong(radix)}, where {@code radix}
     * is the default radix of this scanner.
     *
     * @return the {@code long} scanned from the input
     * @throws InputMismatchException
     *         if the next token does not match the <i>Integer</i>
     *         regular expression, or is out of range
     * @throws NoSuchElementException if input is exhausted
     * @throws IllegalStateException if this scanner is closed
     */
    public long nextLong() {
        return nextLong(defaultRadix);
    }

    /**
     * Scans the next token of the input as a {@code long}.
     * This method will throw {@code InputMismatchException}
     * if the next token cannot be translated into a valid long value as
     * described below. If the translation is successful, the scanner advances
     * past the input that matched.
     *
     * <p> If the next token matches the <a
     * href="#Integer-regex"><i>Integer</i></a> regular expression defined
     * above then the token is converted into a {@code long} value as if by
     * removing all locale specific prefixes, group separators, and locale
     * specific suffixes, then mapping non-ASCII digits into ASCII
     * digits via {@link Character#digit Character.digit}, prepending a
     * negative sign (-) if the locale specific negative prefixes and suffixes
     * were present, and passing the resulting string to
     * {@link Long#parseLong(String, int) Long.parseLong} with the
     * specified radix.
     *
     * <p>If the radix is less than {@link Character#MIN_RADIX Character.MIN_RADIX}
     * or greater than {@link Character#MAX_RADIX Character.MAX_RADIX}, then an
     * {@code IllegalArgumentException} is thrown.
     *
     * @param radix the radix used to interpret the token as an int value
     * @return the {@code long} scanned from the input
     * @throws InputMismatchException
     *         if the next token does not match the <i>Integer</i>
     *         regular expression, or is out of range
     * @throws NoSuchElementException if input is exhausted
     * @throws IllegalStateException if this scanner is closed
     * @throws IllegalArgumentException if the radix is out of range
     */
    public long nextLong(int radix) {
        // Check cached result
        if ((typeCache != null) && (typeCache instanceof Long)
            && this.radix == radix) {
            long val = ((Long)typeCache).longValue();
            useTypeCache();
            return val;
        }
        setRadix(radix);
        clearCaches();
        try {
            String s = next(integerPattern());
            if (matcher.group(SIMPLE_GROUP_INDEX) == null)
                s = processIntegerToken(s);
            return Long.parseLong(s, radix);
        } catch (NumberFormatException nfe) {
            position = matcher.start(); // don't skip bad token
            throw new InputMismatchException(nfe.getMessage());
        }
    }

    /**
     * The float token must be stripped of prefixes, group separators,
     * and suffixes, non ascii digits must be converted into ascii digits
     * before parseFloat will accept it.
     *
     * If there are non-ascii digits in the token these digits must
     * be processed before the token is passed to parseFloat.
     */
    private String processFloatToken(String token) {
        String result = token.replaceAll(groupSeparator, "");
        if (!decimalSeparator.equals("\\."))
            result = result.replaceAll(decimalSeparator, ".");
        boolean isNegative = false;
        int preLen = negativePrefix.length();
        if ((preLen > 0) && result.startsWith(negativePrefix)) {
            isNegative = true;
            result = result.substring(preLen);
        }
        int sufLen = negativeSuffix.length();
        if ((sufLen > 0) && result.endsWith(negativeSuffix)) {
            isNegative = true;
            result = result.substring(result.length() - sufLen,
                                      result.length());
        }
        if (result.equals(nanString))
            result = "NaN";
        if (result.equals(infinityString))
            result = "Infinity";
        if (isNegative)
            result = "-" + result;

        // Translate non-ASCII digits
        Matcher m = NON_ASCII_DIGIT.matcher(result);
        if (m.find()) {
            StringBuilder inASCII = new StringBuilder();
            for (int i=0; i<result.length(); i++) {
                char nextChar = result.charAt(i);
                if (Character.isDigit(nextChar)) {
                    int d = Character.digit(nextChar, 10);
                    if (d != -1)
                        inASCII.append(d);
                    else
                        inASCII.append(nextChar);
                } else {
                    inASCII.append(nextChar);
                }
            }
            result = inASCII.toString();
        }

        return result;
    }

    /**
     * Returns true if the next token in this scanner's input can be
     * interpreted as a float value using the {@link #nextFloat}
     * method. The scanner does not advance past any input.
     *
     * @return true if and only if this scanner's next token is a valid
     *         float value
     * @throws IllegalStateException if this scanner is closed
     */
    public boolean hasNextFloat() {
        setRadix(10);
        boolean result = hasNext(floatPattern());
        if (result) { // Cache it
            try {
                String s = processFloatToken(hasNextResult);
                typeCache = Float.valueOf(Float.parseFloat(s));
            } catch (NumberFormatException nfe) {
                result = false;
            }
        }
        return result;
    }

    /**
     * Scans the next token of the input as a {@code float}.
     * This method will throw {@code InputMismatchException}
     * if the next token cannot be translated into a valid float value as
     * described below. If the translation is successful, the scanner advances
     * past the input that matched.
     *
     * <p> If the next token matches the <a
     * href="#Float-regex"><i>Float</i></a> regular expression defined above
     * then the token is converted into a {@code float} value as if by
     * removing all locale specific prefixes, group separators, and locale
     * specific suffixes, then mapping non-ASCII digits into ASCII
     * digits via {@link Character#digit Character.digit}, prepending a
     * negative sign (-) if the locale specific negative prefixes and suffixes
     * were present, and passing the resulting string to
     * {@link Float#parseFloat Float.parseFloat}. If the token matches
     * the localized NaN or infinity strings, then either "Nan" or "Infinity"
     * is passed to {@link Float#parseFloat(String) Float.parseFloat} as
     * appropriate.
     *
     * @return the {@code float} scanned from the input
     * @throws InputMismatchException
     *         if the next token does not match the <i>Float</i>
     *         regular expression, or is out of range
     * @throws NoSuchElementException if input is exhausted
     * @throws IllegalStateException if this scanner is closed
     */
    public float nextFloat() {
        // Check cached result
        if ((typeCache != null) && (typeCache instanceof Float)) {
            float val = ((Float)typeCache).floatValue();
            useTypeCache();
            return val;
        }
        setRadix(10);
        clearCaches();
        try {
            return Float.parseFloat(processFloatToken(next(floatPattern())));
        } catch (NumberFormatException nfe) {
            position = matcher.start(); // don't skip bad token
            throw new InputMismatchException(nfe.getMessage());
        }
    }

    /**
     * Returns true if the next token in this scanner's input can be
     * interpreted as a double value using the {@link #nextDouble}
     * method. The scanner does not advance past any input.
     *
     * @return true if and only if this scanner's next token is a valid
     *         double value
     * @throws IllegalStateException if this scanner is closed
     */
    public boolean hasNextDouble() {
        setRadix(10);
        boolean result = hasNext(floatPattern());
        if (result) { // Cache it
            try {
                String s = processFloatToken(hasNextResult);
                typeCache = Double.valueOf(Double.parseDouble(s));
            } catch (NumberFormatException nfe) {
                result = false;
            }
        }
        return result;
    }

    /**
     * Scans the next token of the input as a {@code double}.
     * This method will throw {@code InputMismatchException}
     * if the next token cannot be translated into a valid double value.
     * If the translation is successful, the scanner advances past the input
     * that matched.
     *
     * <p> If the next token matches the <a
     * href="#Float-regex"><i>Float</i></a> regular expression defined above
     * then the token is converted into a {@code double} value as if by
     * removing all locale specific prefixes, group separators, and locale
     * specific suffixes, then mapping non-ASCII digits into ASCII
     * digits via {@link Character#digit Character.digit}, prepending a
     * negative sign (-) if the locale specific negative prefixes and suffixes
     * were present, and passing the resulting string to
     * {@link Double#parseDouble Double.parseDouble}. If the token matches
     * the localized NaN or infinity strings, then either "Nan" or "Infinity"
     * is passed to {@link Double#parseDouble(String) Double.parseDouble} as
     * appropriate.
     *
     * @return the {@code double} scanned from the input
     * @throws InputMismatchException
     *         if the next token does not match the <i>Float</i>
     *         regular expression, or is out of range
     * @throws NoSuchElementException if the input is exhausted
     * @throws IllegalStateException if this scanner is closed
     */
    public double nextDouble() {
        // Check cached result
        if ((typeCache != null) && (typeCache instanceof Double)) {
            double val = ((Double)typeCache).doubleValue();
            useTypeCache();
            return val;
        }
        setRadix(10);
        clearCaches();
        // Search for next float
        try {
            return Double.parseDouble(processFloatToken(next(floatPattern())));
        } catch (NumberFormatException nfe) {
            position = matcher.start(); // don't skip bad token
            throw new InputMismatchException(nfe.getMessage());
        }
    }

    // Convenience methods for scanning multi precision numbers

    /**
     * Returns true if the next token in this scanner's input can be
     * interpreted as a {@code BigInteger} in the default radix using the
     * {@link #nextBigInteger} method. The scanner does not advance past any
     * input.
     *
     * @return true if and only if this scanner's next token is a valid
     *         {@code BigInteger}
     * @throws IllegalStateException if this scanner is closed
     */
    public boolean hasNextBigInteger() {
        return hasNextBigInteger(defaultRadix);
    }

    /**
     * Returns true if the next token in this scanner's input can be
     * interpreted as a {@code BigInteger} in the specified radix using
     * the {@link #nextBigInteger} method. The scanner does not advance past
     * any input.
     *
     * <p>If the radix is less than {@link Character#MIN_RADIX Character.MIN_RADIX}
     * or greater than {@link Character#MAX_RADIX Character.MAX_RADIX}, then an
     * {@code IllegalArgumentException} is thrown.
     *
     * @param radix the radix used to interpret the token as an integer
     * @return true if and only if this scanner's next token is a valid
     *         {@code BigInteger}
     * @throws IllegalStateException if this scanner is closed
     * @throws IllegalArgumentException if the radix is out of range
     */
    public boolean hasNextBigInteger(int radix) {
        setRadix(radix);
        boolean result = hasNext(integerPattern());
        if (result) { // Cache it
            try {
                String s = (matcher.group(SIMPLE_GROUP_INDEX) == null) ?
                    processIntegerToken(hasNextResult) :
                    hasNextResult;
                typeCache = new BigInteger(s, radix);
            } catch (NumberFormatException nfe) {
                result = false;
            }
        }
        return result;
    }

    /**
     * Scans the next token of the input as a {@link java.math.BigInteger
     * BigInteger}.
     *
     * <p> An invocation of this method of the form
     * {@code nextBigInteger()} behaves in exactly the same way as the
     * invocation {@code nextBigInteger(radix)}, where {@code radix}
     * is the default radix of this scanner.
     *
     * @return the {@code BigInteger} scanned from the input
     * @throws InputMismatchException
     *         if the next token does not match the <i>Integer</i>
     *         regular expression, or is out of range
     * @throws NoSuchElementException if the input is exhausted
     * @throws IllegalStateException if this scanner is closed
     */
    public BigInteger nextBigInteger() {
        return nextBigInteger(defaultRadix);
    }

    /**
     * Scans the next token of the input as a {@link java.math.BigInteger
     * BigInteger}.
     *
     * <p> If the next token matches the <a
     * href="#Integer-regex"><i>Integer</i></a> regular expression defined
     * above then the token is converted into a {@code BigInteger} value as if
     * by removing all group separators, mapping non-ASCII digits into ASCII
     * digits via the {@link Character#digit Character.digit}, and passing the
     * resulting string to the {@link
     * java.math.BigInteger#BigInteger(java.lang.String)
     * BigInteger(String, int)} constructor with the specified radix.
     *
     * <p>If the radix is less than {@link Character#MIN_RADIX Character.MIN_RADIX}
     * or greater than {@link Character#MAX_RADIX Character.MAX_RADIX}, then an
     * {@code IllegalArgumentException} is thrown.
     *
     * @param radix the radix used to interpret the token
     * @return the {@code BigInteger} scanned from the input
     * @throws InputMismatchException
     *         if the next token does not match the <i>Integer</i>
     *         regular expression, or is out of range
     * @throws NoSuchElementException if the input is exhausted
     * @throws IllegalStateException if this scanner is closed
     * @throws IllegalArgumentException if the radix is out of range
     */
    public BigInteger nextBigInteger(int radix) {
        // Check cached result
        if ((typeCache != null) && (typeCache instanceof BigInteger)
            && this.radix == radix) {
            BigInteger val = (BigInteger)typeCache;
            useTypeCache();
            return val;
        }
        setRadix(radix);
        clearCaches();
        // Search for next int
        try {
            String s = next(integerPattern());
            if (matcher.group(SIMPLE_GROUP_INDEX) == null)
                s = processIntegerToken(s);
            return new BigInteger(s, radix);
        } catch (NumberFormatException nfe) {
            position = matcher.start(); // don't skip bad token
            throw new InputMismatchException(nfe.getMessage());
        }
    }

    /**
     * Returns true if the next token in this scanner's input can be
     * interpreted as a {@code BigDecimal} using the
     * {@link #nextBigDecimal} method. The scanner does not advance past any
     * input.
     *
     * @return true if and only if this scanner's next token is a valid
     *         {@code BigDecimal}
     * @throws IllegalStateException if this scanner is closed
     */
    public boolean hasNextBigDecimal() {
        setRadix(10);
        boolean result = hasNext(decimalPattern());
        if (result) { // Cache it
            try {
                String s = processFloatToken(hasNextResult);
                typeCache = new BigDecimal(s);
            } catch (NumberFormatException nfe) {
                result = false;
            }
        }
        return result;
    }

    /**
     * Scans the next token of the input as a {@link java.math.BigDecimal
     * BigDecimal}.
     *
     * <p> If the next token matches the <a
     * href="#Decimal-regex"><i>Decimal</i></a> regular expression defined
     * above then the token is converted into a {@code BigDecimal} value as if
     * by removing all group separators, mapping non-ASCII digits into ASCII
     * digits via the {@link Character#digit Character.digit}, and passing the
     * resulting string to the {@link
     * java.math.BigDecimal#BigDecimal(java.lang.String) BigDecimal(String)}
     * constructor.
     *
     * @return the {@code BigDecimal} scanned from the input
     * @throws InputMismatchException
     *         if the next token does not match the <i>Decimal</i>
     *         regular expression, or is out of range
     * @throws NoSuchElementException if the input is exhausted
     * @throws IllegalStateException if this scanner is closed
     */
    public BigDecimal nextBigDecimal() {
        // Check cached result
        if ((typeCache != null) && (typeCache instanceof BigDecimal)) {
            BigDecimal val = (BigDecimal)typeCache;
            useTypeCache();
            return val;
        }
        setRadix(10);
        clearCaches();
        // Search for next float
        try {
            String s = processFloatToken(next(decimalPattern()));
            return new BigDecimal(s);
        } catch (NumberFormatException nfe) {
            position = matcher.start(); // don't skip bad token
            throw new InputMismatchException(nfe.getMessage());
        }
    }

    /**
     * Resets this scanner.
     *
     * <p> Resetting a scanner discards all of its explicit state
     * information which may have been changed by invocations of
     * {@link #useDelimiter useDelimiter()},
     * {@link #useLocale useLocale()}, or
     * {@link #useRadix useRadix()}.
     *
     * <p> An invocation of this method of the form
     * {@code scanner.reset()} behaves in exactly the same way as the
     * invocation
     *
     * <blockquote><pre>{@code
     *   scanner.useDelimiter("\\p{javaWhitespace}+")
     *          .useLocale(Locale.getDefault(Locale.Category.FORMAT))
     *          .useRadix(10);
     * }</pre></blockquote>
     *
     * @return this scanner
     *
     * @since 1.6
     */
    public Scanner reset() {
        delimPattern = WHITESPACE_PATTERN;
        useLocale(Locale.getDefault(Locale.Category.FORMAT));
        useRadix(10);
        clearCaches();
        modCount++;
        return this;
    }

    /**
     * Returns a stream of delimiter-separated tokens from this scanner. The
     * stream contains the same tokens that would be returned, starting from
     * this scanner's current state, by calling the {@link #next} method
     * repeatedly until the {@link #hasNext} method returns false.
     *
     * <p>The resulting stream is sequential and ordered. All stream elements are
     * non-null.
     *
     * <p>Scanning starts upon initiation of the terminal stream operation, using the
     * current state of this scanner. Subsequent calls to any methods on this scanner
     * other than {@link #close} and {@link #ioException} may return undefined results
     * or may cause undefined effects on the returned stream. The returned stream's source
     * {@code Spliterator} is <em>fail-fast</em> and will, on a best-effort basis, throw a
     * {@link java.util.ConcurrentModificationException} if any such calls are detected
     * during stream pipeline execution.
     *
     * <p>After stream pipeline execution completes, this scanner is left in an indeterminate
     * state and cannot be reused.
     *
     * <p>If this scanner contains a resource that must be released, this scanner
     * should be closed, either by calling its {@link #close} method, or by
     * closing the returned stream. Closing the stream will close the underlying scanner.
     * {@code IllegalStateException} is thrown if the scanner has been closed when this
     * method is called, or if this scanner is closed during stream pipeline execution.
     *
     * <p>This method might block waiting for more input.
     *
     * @apiNote
     * For example, the following code will create a list of
     * comma-delimited tokens from a string:
     *
     * <pre>{@code
     * List<String> result = new Scanner("abc,def,,ghi")
     *     .useDelimiter(",")
     *     .tokens()
     *     .collect(Collectors.toList());
     * }</pre>
     *
     * <p>The resulting list would contain {@code "abc"}, {@code "def"},
     * the empty string, and {@code "ghi"}.
     *
     * @return a sequential stream of token strings
     * @throws IllegalStateException if this scanner is closed
     * @since 9
     */
    public Stream<String> tokens() {
        ensureOpen();
        Stream<String> stream = StreamSupport.stream(new TokenSpliterator(), false);
        return stream.onClose(this::close);
    }

    class TokenSpliterator extends Spliterators.AbstractSpliterator<String> {
        int expectedCount = -1;

        TokenSpliterator() {
            super(Long.MAX_VALUE,
                  Spliterator.IMMUTABLE | Spliterator.NONNULL | Spliterator.ORDERED);
        }

        @Override
        public boolean tryAdvance(Consumer<? super String> cons) {
            if (expectedCount >= 0 && expectedCount != modCount) {
                throw new ConcurrentModificationException();
            }

            if (hasNext()) {
                String token = next();
                expectedCount = modCount;
                cons.accept(token);
                if (expectedCount != modCount) {
                    throw new ConcurrentModificationException();
                }
                return true;
            } else {
                expectedCount = modCount;
                return false;
            }
        }
    }

    /**
     * Returns a stream of match results from this scanner. The stream
     * contains the same results in the same order that would be returned by
     * calling {@code findWithinHorizon(pattern, 0)} and then {@link #match}
     * successively as long as {@link #findWithinHorizon findWithinHorizon()}
     * finds matches.
     *
     * <p>The resulting stream is sequential and ordered. All stream elements are
     * non-null.
     *
     * <p>Scanning starts upon initiation of the terminal stream operation, using the
     * current state of this scanner. Subsequent calls to any methods on this scanner
     * other than {@link #close} and {@link #ioException} may return undefined results
     * or may cause undefined effects on the returned stream. The returned stream's source
     * {@code Spliterator} is <em>fail-fast</em> and will, on a best-effort basis, throw a
     * {@link java.util.ConcurrentModificationException} if any such calls are detected
     * during stream pipeline execution.
     *
     * <p>After stream pipeline execution completes, this scanner is left in an indeterminate
     * state and cannot be reused.
     *
     * <p>If this scanner contains a resource that must be released, this scanner
     * should be closed, either by calling its {@link #close} method, or by
     * closing the returned stream. Closing the stream will close the underlying scanner.
     * {@code IllegalStateException} is thrown if the scanner has been closed when this
     * method is called, or if this scanner is closed during stream pipeline execution.
     *
     * <p>As with the {@link #findWithinHorizon findWithinHorizon()} methods, this method
     * might block waiting for additional input, and it might buffer an unbounded amount of
     * input searching for a match.
     *
     * @apiNote
     * For example, the following code will read a file and return a list
     * of all sequences of characters consisting of seven or more Latin capital
     * letters:
     *
     * <pre>{@code
     * try (Scanner sc = new Scanner(Path.of("input.txt"))) {
     *     Pattern pat = Pattern.compile("[A-Z]{7,}");
     *     List<String> capWords = sc.findAll(pat)
     *                               .map(MatchResult::group)
     *                               .collect(Collectors.toList());
     * }
     * }</pre>
     *
     * @param pattern the pattern to be matched
     * @return a sequential stream of match results
     * @throws NullPointerException if pattern is null
     * @throws IllegalStateException if this scanner is closed
     * @since 9
     */
    public Stream<MatchResult> findAll(Pattern pattern) {
        Objects.requireNonNull(pattern);
        ensureOpen();
        Stream<MatchResult> stream = StreamSupport.stream(new FindSpliterator(pattern), false);
        return stream.onClose(this::close);
    }

    /**
     * Returns a stream of match results that match the provided pattern string.
     * The effect is equivalent to the following code:
     *
     * <pre>{@code
     *     scanner.findAll(Pattern.compile(patString))
     * }</pre>
     *
     * @param patString the pattern string
     * @return a sequential stream of match results
     * @throws NullPointerException if patString is null
     * @throws IllegalStateException if this scanner is closed
     * @throws PatternSyntaxException if the regular expression's syntax is invalid
     * @since 9
     * @see java.util.regex.Pattern
     */
    public Stream<MatchResult> findAll(String patString) {
        Objects.requireNonNull(patString);
        ensureOpen();
        return findAll(patternCache.forName(patString));
    }

    class FindSpliterator extends Spliterators.AbstractSpliterator<MatchResult> {
        final Pattern pattern;
        int expectedCount = -1;
        private boolean advance = false; // true if we need to auto-advance

        FindSpliterator(Pattern pattern) {
            super(Long.MAX_VALUE,
                  Spliterator.IMMUTABLE | Spliterator.NONNULL | Spliterator.ORDERED);
            this.pattern = pattern;
        }

        @Override
        public boolean tryAdvance(Consumer<? super MatchResult> cons) {
            ensureOpen();
            if (expectedCount >= 0) {
                if (expectedCount != modCount) {
                    throw new ConcurrentModificationException();
                }
            } else {
                // init
                matchValid = false;
                matcher.usePattern(pattern);
                expectedCount = modCount;
            }

            while (true) {
                // assert expectedCount == modCount
                if (nextInBuffer()) { // doesn't increment modCount
                    cons.accept(matcher.toMatchResult());
                    if (expectedCount != modCount) {
                        throw new ConcurrentModificationException();
                    }
                    return true;
                }
                if (needInput)
                    readInput(); // doesn't increment modCount
                else
                    return false; // reached end of input
            }
        }

        // reimplementation of findPatternInBuffer with auto-advance on zero-length matches
        private boolean nextInBuffer() {
            if (advance) {
                if (position + 1 > buf.limit()) {
                    if (!sourceClosed)
                        needInput = true;
                    return false;
                }
                position++;
                advance = false;
            }
            matcher.region(position, buf.limit());
            if (matcher.find() && (!matcher.hitEnd() || sourceClosed)) {
                 // Did not hit end, or hit real end
                 position = matcher.end();
                 advance = matcher.start() == position;
                 return true;
            }
            if (!sourceClosed)
                needInput = true;
            return false;
        }
    }

    /** Small LRU cache of Patterns. */
    private static class PatternLRUCache {

        private Pattern[] oa = null;
        private final int size;

        PatternLRUCache(int size) {
            this.size = size;
        }

        boolean hasName(Pattern p, String s) {
            return p.pattern().equals(s);
        }

        void moveToFront(Object[] oa, int i) {
            Object ob = oa[i];
            for (int j = i; j > 0; j--)
                oa[j] = oa[j - 1];
            oa[0] = ob;
        }

        Pattern forName(String name) {
            if (oa == null) {
                Pattern[] temp = new Pattern[size];
                oa = temp;
            } else {
                for (int i = 0; i < oa.length; i++) {
                    Pattern ob = oa[i];
                    if (ob == null)
                        continue;
                    if (hasName(ob, name)) {
                        if (i > 0)
                            moveToFront(oa, i);
                        return ob;
                    }
                }
            }

            // Create a new object
            Pattern ob = Pattern.compile(name);
            oa[oa.length - 1] = ob;
            moveToFront(oa, oa.length - 1);
            return ob;
        }
    }
}
