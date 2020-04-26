# AB工具类Java版本实现


```java
import org.apache.commons.math3.distribution.NormalDistribution;

import java.math.BigDecimal;
import java.math.MathContext;
import java.math.RoundingMode;
import java.util.ArrayList;
import java.util.List;

/**
 * 描述: AB分析工具类
 *
 * @author zhouraohust
 * @create 2020-04-26 2:40 下午
 */
public class AlalysisUtils {

    private static NormalDistribution nd = new NormalDistribution();

    private final static int SCALE = 16;

    //采用双边检测
    private static final double FIRST_TYPE_ERROR_CHANCE = 0.05;

    public static double GetStatisticalPower(BigDecimal expCt, BigDecimal controlCt, BigDecimal expCnt, BigDecimal controlCnt) {
        if (BigDecimal.ZERO.equals(expCt) && BigDecimal.ZERO.equals(controlCt)) {
            return 0;
        }
        BigDecimal zScore = GetZScore(expCt, controlCt, expCnt, controlCnt);
        BigDecimal staticZScore = new BigDecimal(nd.inverseCumulativeProbability(1 - FIRST_TYPE_ERROR_CHANCE / 2));

        return 1 - nd.cumulativeProbability(staticZScore.subtract(zScore).doubleValue()) + nd.cumulativeProbability(staticZScore.multiply(new BigDecimal(-1)).subtract(zScore).doubleValue());
    }

    public static List<BigDecimal> GetConfidenceInterval(BigDecimal expCt, BigDecimal controlCt, BigDecimal expCnt, BigDecimal controlCnt) {
        List<BigDecimal> result = new ArrayList<>();
        if (BigDecimal.ZERO.equals(expCt) && BigDecimal.ZERO.equals(controlCt)) {
            result.add(BigDecimal.ZERO);
            result.add(BigDecimal.ZERO);
            return result;
        }
        BigDecimal expRatio = expCt.divide(expCnt, SCALE, BigDecimal.ROUND_HALF_UP);
        BigDecimal controlRatio = controlCt.divide(controlCnt, SCALE, BigDecimal.ROUND_HALF_UP);
        BigDecimal variance = getVariance(expRatio, controlRatio, expCnt, controlCnt);
        double v = nd.inverseCumulativeProbability(FIRST_TYPE_ERROR_CHANCE / 2);
        BigDecimal wave = variance.multiply(new BigDecimal(v)).abs();
        result.add(expRatio.subtract(controlRatio).subtract(wave));
        result.add(expRatio.subtract(controlRatio).add(wave));
        return result;
    }

    public static double GetPValue(BigDecimal expCt, BigDecimal controlCt, BigDecimal expCnt, BigDecimal controlCnt) {
        if (BigDecimal.ZERO.equals(expCt) && BigDecimal.ZERO.equals(controlCt)) {
            return 1;
        }
        BigDecimal zScore = GetZScore(expCt, controlCt, expCnt, controlCnt);
        return 1 - nd.cumulativeProbability(zScore.doubleValue());
    }

    private static BigDecimal GetZScore(BigDecimal expCt, BigDecimal controlCt, BigDecimal expCnt, BigDecimal controlCnt) {
        BigDecimal expRatio = expCt.divide(expCnt, SCALE, BigDecimal.ROUND_HALF_UP);
        BigDecimal controlRatio = controlCt.divide(controlCnt, SCALE, BigDecimal.ROUND_HALF_UP);
        BigDecimal variance = getVariance(expRatio, controlRatio, expCnt, controlCnt);
        return (expRatio.subtract(controlRatio)).divide(variance, SCALE, BigDecimal.ROUND_HALF_UP).abs();
    }

    private static BigDecimal getVariance(BigDecimal expRatio, BigDecimal controlRatio, BigDecimal expCnt, BigDecimal controlCnt) {
        BigDecimal se_experiment = expRatio.multiply(BigDecimal.ONE.subtract(expRatio)).divide(expCnt, SCALE, BigDecimal.ROUND_HALF_UP);
        BigDecimal se_control = controlRatio.multiply(BigDecimal.ONE.subtract(controlRatio)).divide(controlCnt, SCALE, BigDecimal.ROUND_HALF_UP);
        return sqrt(se_experiment.add(se_control));
    }

    private static BigDecimal sqrt(BigDecimal value) {
        BigDecimal num2 = BigDecimal.valueOf(2);
        int precision = 100;
        MathContext mc = new MathContext(precision, RoundingMode.HALF_UP);
        BigDecimal deviation = value;
        int cnt = 0;
        while (cnt < precision) {
            deviation = (deviation.add(value.divide(deviation, mc))).divide(num2, mc);
            cnt++;
        }
        deviation = deviation.setScale(SCALE, BigDecimal.ROUND_HALF_UP);
        return deviation;
    }
}
```