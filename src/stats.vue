<template>
<div id="stats">
  <div class="stat signal_strategy">
    <span class="label">Signal Strategy</span><span class="value">MACD 12/26/9</span>
  </div>
  <div class="stat annualized_return">
    <span class="label"><span class="period">{{ shared.period }}</span> Annualized</span> <span class="value" v-bind:class="portfolio_annualized_change_class">{{ formatted_portfolio_annualized_change }} ({{ portfolio_percent_change_annualized }}%)</span>
  </div>
  <div class="stat max_drawdown">
    <span class="label">Max Drawdown</span><span class="value">{{ formatted_max_drawdown }}%</span>
  </div>
  <div class="stat gain_to_pain_ratio">
    <span class="label">Gain / Pain Ratio</span><span class="value">{{ gain_to_pain_ratio }}</span>
  </div>
  <div class="stat mean_allocation">
    <span class="label">Mean Allocation</span><span class="value">{{ mean_allocation }}%</span>
  </div>
</div>
</template>

<script>
import store from './store'
import analytics from "portfolio-analytics"

var currency_formatter = new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
});

export default {
  name: 'stats',
  data() {
    return {
      shared: store.state
    }
  },
  computed: {
    mean_allocation: function () {
      let self = this;
      let result = 0.0;
      let share_percentages = this.shared.positions.map(function(position) {
        return (position.holding_value / self.shared.raw_portfolio_value) * 100;
      });
      let sum = share_percentages.reduce(function(accumulator, currentValue) {
        return accumulator + currentValue;
      });

      result = sum / share_percentages.length;

      return Math.round(result * 100) / 100;
    },
    formatted_portfolio_annualized_change: function () {
      let amount = this.portfolio_change_annualized;
      let sign = "";

      if (amount > 0) {
        sign = "+"
      }

      return sign + currency_formatter.format(amount);
    },
    formatted_max_drawdown: function () {
      let amount = this.max_drawdown;
      let sign = "";

      if (amount > 0) {
        sign = "+"
      }

      return sign + amount;
    },
    portfolio_annualized_change_class: function() {
      return {
        is_positive: (this.portfolio_percent_change_annualized > 0),
        is_neutral: (this.portfolio_percent_change_annualized == 0),
        is_negative: (this.portfolio_percent_change_annualized < 0)
      };
    },
    portfolio_percent_change: function () {
      let result = (this.shared.raw_portfolio_change / this.shared.raw_portfolio_value) * 100;
      return result;
    },
    portfolio_percent_change_annualized: function() {
      let result = this.portfolio_percent_change;
      let period = this.shared.period;

      if (period == "1d") {
        result = this.portfolio_percent_change * 365;
      } else if (period == "1w") {
        result = this.portfolio_percent_change * 52;
      } else if (period == "1m") {
        result = this.portfolio_percent_change * 12;
      } else if (period == "3m") {
        result = this.portfolio_percent_change * 4;
      } else if (period == "1y") {
        result = this.portfolio_percent_change * 1;
      } else if (period == "all") {
        result = this.portfolio_percent_change / 5;
      }

      return Math.round(result * 100) / 100;
    },
    portfolio_change_annualized: function() {
      let result = this.portfolio_percent_change;
      let period = this.shared.period;

      if (period == "1d") {
        result = this.shared.raw_portfolio_change * 365;
      } else if (period == "1w") {
        result = this.shared.raw_portfolio_change * 52;
      } else if (period == "1m") {
        result = this.shared.raw_portfolio_change * 12;
      } else if (period == "3m") {
        result = this.shared.raw_portfolio_change * 4;
      } else if (period == "1y") {
        result = this.shared.raw_portfolio_change * 1;
      } else if (period == "all") {
        result = this.shared.raw_portfolio_change / 5;
      }

      return Math.round(result * 100) / 100;
    },
    max_drawdown: function() {
      let values = this.shared.portfolio_historical_values;
      let result = -analytics.maxDrawdown(values);

      return Math.round(result * 10000) / 100;
    },
    gain_to_pain_ratio: function() {
      let values = this.shared.portfolio_historical_values;
      let result = analytics.gainToPainRatio(values);

      return Math.round(result * 100) / 100;
    }
  }
}
</script>

<style lang="scss">

div#stats {
  box-sizing: border-box;
  margin: 15px 0;
  padding: 0 0 0 15px;
  font-size: 0.75em;
  width: 25%;

  @media (max-width: 576px) {
    margin: 0 0 25px;
    order: 1;
    padding: 0 15px;
    width: 100%;
  }

  .stat {
    display: flex;
    margin-bottom: 0.75em;

    span.label {
      font-weight: bold;
      text-align: left;
      width: 50%;
    }

    span.value {
      text-align: right;
      width: 50%;
    }
  }

  .max_drawdown {
    span.value {
      &.is_positive {
        color: #98c379;
      }

      &.is_neutral {
        color: #e5c07b;
      }

      &.is_negative {
        color: #e05252;
      }
    }
  }

  .annualized_return {
    span.period {
      text-transform: uppercase;
    }

    span.value {
      &.is_positive {
        color: #98c379;
      }

      &.is_neutral {
        color: #e5c07b;
      }

      &.is_negative {
        color: #e05252;
      }
    }
  }
}
</style>
